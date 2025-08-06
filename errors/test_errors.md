## If tests failed: 
export { ApiError } from './core/ApiError';
SyntaxError: Unexpected token 'export'
## Solution
Add @procore/document-service-api|@procore/documents-shared-components to the procore.config.js  => defaultConfig.transformIgnorePatterns

## If tests failed:
TestingLibraryElementError: Unable to find an element with the text: stubbed. This could be because the text is broken up by multiple elements. In this case, you can provide a function for your text matcher to make your matcher more flexible.

## Solution
Add this mock for the tests jest mock server
jest.mock('./useCDNFeatureFlag', () => ({
  useCDNFeatureFlag: () => ({ data: false }),
}));
OR
jest.mock('@procore/web-sdk-feature-toggle', () => ({
  useFeatureFlag: () => false,
  FeatureToggleProvider: ({ children }: { children: React.ReactNode }) => (
    <>{children}</>
  ),
}));
MONDATORY: YOU SHALL NOT MOCK CDN TRANSLATION ITSELF.

## If test failed
TypeError: reactResizeDetector.useResizeDetector is not a function
## Solution
Add src/mocks/react-resize-detector.jswith this content 
/* global jest */
module.exports = {
  useResizeDetector: jest.fn(() => ({ width: 100, height: 100, ref: {} })),
};