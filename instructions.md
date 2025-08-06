# Automated Repository Migration Pipeline

## General
1. You need to make sure that before you are plan changes or do them in loop you are follow by instructions and you are not getting up with something by your memory 

## Mission

1.  **Migrate** the translation delivery process to use the `@procore/cdn-translations` package, conditionally loading translations via a LaunchDarkly feature flag across all repositories in `./repositories`.
2.  **Validate** all changes by running linters and tests, ensuring they pass successfully.
3.  **Log** all errors encountered during linters and tests.
4.  **Summarize** the process and any applied fixes for future reference.

## Workflow

1.  **Planning:**
    -   Based on the `## Technical Instructions`, `## Code Style Changes`, and the project structure, generate a step-by-step plan for the entire migration process.
2.  **Approval:**
    -   Present the generated plan to the user for approval. Do not proceed until explicit approval is given.
3.  **Execution:**
    -   Once approved, execute the plan's steps sequentially on each repository located in `./repositories`.
4.  **Validation:**
    -   After applying changes to a repository, execute: yarn run lint, yarn run test, yarn run build. MONDAORY: MAKE SURE ALL TESTS ARE PASSED ON.
5. **Post cleanup:**
    – Make sure that any unused imports are deleted
6ю **Outcome:**
    – Do not generate outcome documentation, just short summary 

## Code Style

1.  **Determine Rules:**
    -  Ensure all subsequent code changes adhere to the linter rules.

## Error Handling

1.  **MANDATORY: Search Existing Solutions First:**
    -   **BEFORE attempting any fix**, you MUST search ALL `.md` files in the `./errors` directory.
    -   Read the full content of each file to identify matching error patterns, not just filenames.
    -   Look for similar keywords: error messages, package names, component names, error types.
    -   Check for partial matches - existing solutions may apply to related issues.

2.  **Apply Existing Fixes:**
    -   If a matching or similar error description is found, apply the prescribed fix EXACTLY as documented.
    -   Adapt the existing solution to your specific case if needed (e.g., different package versions).
    -   Do NOT create new documentation for existing error patterns.

3.  **Independent Resolution (Only if no existing solution):**
    -   If no related information is found after thorough search, resolve the problem independently.
    -   Document your search process: mention which existing errors you checked.

4.  **New Documentation (Only after successful completion):**
    -   Create new Markdown files in `./errors` directory ONLY after all validation and build commands pass.
    -   Before creating new files, verify no duplication with existing content.
    -   Use consistent format: Error Description, Affected Files, Solution Applied, Core Parameters, Background.
    -   Include specific package versions, repo-name, and error context.

5.  **Duplication Prevention:**
    -   Check file sizes - empty files (0 bytes) may be placeholders that need content.
    -   Consolidate similar errors rather than creating multiple files.
    -   Reference existing solutions in new documentation when applicable.

## Technical Instructions for Microfrontend

#### **1. Package Updates**

1.  **Update Existing Packages:**
    -   Read the root `packages.json` file to get the list of "core" dependency versions.
    -   For each repository in `./repositories`, iterate through the dependencies in its `package.json`.
    -   If a dependency in a repository's `package.json` also exists in the root `packages.json`, update its version to match the version specified in the root file.
    -   **Do not** add any new dependencies to the repository's `package.json` that are not already present.

2.  **Add New Package:**
    -   Add `@procore/cdn-translations` to the `dependencies` list in each repository's `package.json` file. The version should be taken from the "Centralized localization Migration guide."
    -   Upgrade `core-react` to the version specified in the same guide.
    -   Run the package manager's install command to update dependencies.

#### **2. Feature Flag Integration**

1.  **Identify Usage:**
    -   Search the codebase for the usage of `I18nProvider` or `useI18n`.
2.  **Locate Feature Flag:**
    -   The flag name is defined by the `CDN_TRANSLATION_FEATURE_FLAG_KEY` variable from `@procore/cdn-translations`. The `launchDarklyProjectId` can be imported from the same package using `getCDNTranslationLDId()`.
3.  **Conditional Wrapping:**
    -   **Condition 2.1: If `@procore/web-sdk-feature-toggle` is already in use:**
        -   Wrap the `I18nProvider` component with a new `FeatureToggleProvider`.
        -   The new provider should be a direct child of `ThirdPartyScriptsProvider` and `PageErrorBoundaryProvider`.

        ```jsx
        <ThirdPartyScripts ...>
          <PageErrorBoundaryProvider ...>
            <FeatureToggleProvider
              apiKey={getCDNTranslationLDId(window.location.hostname)}
              context={{
                user: {
                  companyId: params?.companyId,
                  projectId: params?.projectId,
                  locale: environmentContext?.i18n?.locale ?? 'en',
                  email: environmentContext?.user?.login ?? '',
                },
              }}
              waitTillReady
            >
            {/* ... other providers and components ... */}
            </FeatureToggleProvider>
          </PageErrorBoundaryProvider>
        </ThirdPartyScripts>
        ```

    -   **Condition 2.2: If `I18nProvider` is already wrapped by another `FeatureToggleProvider` with a different `projectId`:**
        -   Move the existing `FeatureToggleProvider` to be a child component of the new `I18nProvider`.

        **Before:**
        ```jsx
        <FeatureToggleProvider apiKey="otherApiKey" ... >
          <I18nProvider .. >
          . . .
          </I18nProvider>
        </FeatureToggleProvider>
        ```

        **After:**
        ```jsx
        <FeatureToggleProvider apiKey={getCDNTranslationLDId(window.location.hostname)} ... >
          <I18nProvider .. >
            <FeatureToggleProvider apiKey="otherApiKey" ... >
            . . .
            </FeatureToggleProvider>
          </I18nProvider>
        </FeatureToggleProvider>
        ```
4.  **Consume the Flag:**
    -   Inside the `I18nProvider` component, add the following line to consume the feature flag:

    ```javascript
    const isCDNEnabled = useFeatureFlag(CDN_TRANSLATION_FEATURE_FLAG_KEY, false);
    ```

#### **3. Implement `useRequestTranslations` Hook**

1.  Add the `useRequestTranslations` hook to the component. Configure the hook to determine the translation type (file or folder-based), the absolute file path, and the user's locale.

    ```javascript
    const cdnTranslations = useRequestTranslations({
        type: "file", // determined if the package is using file or folder based locales
        absolute_file_path: (locale) => `repo-name/file-path/${locale}.json`, // add path to the locale file in the repo
        locale: env?.metadata?.i18n?.locale, // user locale without passing it on getOverrideLocale or any other function.
      },
      {
        en, // default translations and translations that will not be uploaded to the CDN
      },
      {
        oldTranslations: translations, // translations was originally passed to the I18nProvider or useI18n
        enableCDN: isCDNEnabled, // enableCDN flag to determine if the hook should return the old translations or the CDN translations
      }
    );
    ```

    -   **Path for folder-based locales:**
        ```javascript
        absolute_file_path: (locale, file_name) => `repo-name/folder-path/${locale}/${file_name}.json`
        ```
    -   **Example for non-standard locales:**
        ```javascript
        const cdnTranslations = useRequestTranslations({
            type: "file",
            absolute_file_path: (locale) => `repo-name/file-path/${locale}`,
            locale: env?.metadata?.i18n?.locale,
          },
          {
            en,
            pseudo: pseudo,
            "en-owner": en-owner,
            "en-budget": en-budget,
          },
          {
            oldTranslations: translations,
            enableCDN: isCDNEnabled,
          }
        );
        ```

#### **4. ONLY If code base has ApplicationLoader component used Modify Loading Condition, otherwise skip this**

-   **Condition: If `@web-sdk/feature-toggle` is used:**
    ```javascript
    if (cdnTranslations.status === 'pending') {
        return <ApplicationLoader />;
    }
    ```
-   **Condition: If `react-query` is used:**
    ```javascript
    if (isPending || cdnTranslations.status === 'pending') {
        return <ApplicationLoader />;
    }
    ```

#### **5. Update `I18nProvider` and `useI18n`**

-   Update the `translations` prop of the `I18nProvider` component to `cdnTranslations.translations`.
-   Add the `enableCDN` prop and set it to `isCDNEnabled`. This applies to `useI18n` as well.

    ```jsx
    <I18nProvider
          locale={env?.metadata?.i18n?.locale}
          translations={cdnTranslations.translations}
          enableCDN={isCDNEnabled}
    >
    ```

    ```javascript
    useI18n({
        locale: env?.metadata?.i18n?.locale,
        translations: cdnTranslations.translations,
        enableCDN: isCDNEnabled,
    })
    ```

#### **6. Mock Server Update**

-   Add the following line to the mock server configuration:

    ```javascript
    this.passthrough('[https://translations.cdn.procoretech-qa.com/](https://translations.cdn.procoretech-qa.com/)**');
    ```