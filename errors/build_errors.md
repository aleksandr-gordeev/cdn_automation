
## Error: TS2307: Cannot find module '@procore/labs-file-select/dist/types/constants' or its corresponding type declarations.
import { AppEnvironment } from '@procore/labs-file-select/dist/types/constants';

## Solution: Replace AppEnvironment with ConnectedFileSelectProps['environment']
import { ConnectedFileSelectProps } from '@procore/labs-file-select';


## Error: TS2307: Cannot find module '@procore/labs-file-select/dist/types/Attachments/AttachmentManager' or its corresponding type declarations.
import { AttachmentItemProps } from '@procore/labs-file-select/dist/types/Attachments/AttachmentManager';


## Solution:
import { AttachmentItemProps } from '@procore/labs-file-select';