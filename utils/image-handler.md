---
name: image-handler
category: utils
stack: [expo-image-picker, react-native, typescript]
keywords: [image-picker, ios, android, multiselect, normalization]
source-files: [libs/utils/imageHandlers.ts]
---

# Image Handler

## Problem
You need to pick images from device library with cross-platform normalization (iOS path handling, type detection) and support for single or multiple selection.

## When to Use
- Building image upload features
- Normalizing image URIs across iOS and Android
- Supporting both single and multiple image selection
- Validating image size before upload

## Implementation

### Dependencies
```json
{
  "expo-image-picker": "^15.0.0"
}
```

### Code

```typescript
import * as ImagePicker from 'expo-image-picker';
import { Platform } from 'react-native';

export const MAX_SIZE_IN_BYTES = 5 * 1024 * 1024; // 5MB

export type TPickedImageAsset = {
  uri: string;
  type?: string;
  size?: number;
  name: string;
};

type PickImageHandlerArgs = {
  allowsMultipleSelection?: boolean;
  mediaTypes?: ImagePicker.MediaType;
};

const formattedAssetFiles = (item: ImagePicker.ImagePickerAsset): TPickedImageAsset => {
  const filename = item.uri.split('/').pop() || 'image.jpg';
  const match = /\.(\w+)$/.exec(filename);
  const type = match ? `image/${match[1]}` : 'image/jpeg';

  return {
    size: item.fileSize,
    // iOS: remove 'file://' prefix; Android keeps full URI
    uri: Platform.OS === 'ios' ? item.uri.replace('file://', '') : item.uri,
    name: filename,
    type: type,
  };
};

export const pickImageHandler = async ({
  allowsMultipleSelection = false,
  mediaTypes,
}: PickImageHandlerArgs): Promise<TPickedImageAsset[] | undefined> => {
  const result = await ImagePicker.launchImageLibraryAsync({
    mediaTypes: mediaTypes || ImagePicker.MediaTypeOptions.Images,
    allowsEditing: false,
    quality: 1,
    legacy: true,
    selectionLimit: 4,
    allowsMultipleSelection,
  });

  if (!result.canceled) {
    return result.assets.map((item) => formattedAssetFiles(item));
  }
};
```

## Usage Example

```typescript
import { pickImageHandler } from 'libs/utils';

const handlePickImage = async () => {
  const images = await pickImageHandler({ allowsMultipleSelection: false });
  if (images && images.length > 0) {
    console.log('Selected image:', images[0].uri, images[0].type);
    // Upload images[0]
  }
};
```

## Gotchas

- **iOS path prefix**: iOS returns `file://` prefix; remove it for FormData. Android doesn't.
- **Type detection**: Relies on filename extension. If extension is missing, defaults to `image/jpeg`.
- **File size**: Check `fileSize` before upload if you have size limits.
- **Permissions**: Request photo library permissions before calling. Use `expo-image-picker` requestPermissions async.
