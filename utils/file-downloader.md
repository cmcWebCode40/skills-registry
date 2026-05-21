---
name: file-downloader
category: utils
stack: [expo-file-system, expo-media-library, react-native, typescript]
keywords: [file-download, permissions, media-library, ios-android, redirect-handling]
source-files: [libs/utils/fileDownloader.ts]
---

# File Downloader

## Problem
You need to download images from URLs with redirect handling, permission checks, and save to device photo library with cross-platform support.

## When to Use
- Building image download/save features
- Handling HTTP redirects during download
- Validating content type before saving
- Organizing downloads into albums (iOS)

## Implementation

### Dependencies
```json
{
  "expo-file-system": "^16.0.0",
  "expo-media-library": "^15.0.0"
}
```

### Code

```typescript
import * as FileSystem from 'expo-file-system/legacy';
import * as MediaLibrary from 'expo-media-library';
import { Platform } from 'react-native';
import Toast from 'react-native-toast-message';

const getFinalImageUrl = async (url: string): Promise<string> => {
  try {
    const response = await fetch(url, {
      method: 'GET',
      redirect: 'follow',
    });
    return response.url; // Follow redirects
  } catch {
    return url; // Fallback to original URL
  }
};

export const downloadImage = async (
  imageUrl: string,
  fileName?: string,
  showAlerts: boolean = true
): Promise<boolean> => {
  try {
    if (!imageUrl || imageUrl.trim() === '') {
      if (showAlerts) {
        Toast.show({
          type: 'error',
          text1: 'Error',
          text2: 'Invalid image URL provided.',
        });
      }
      return false;
    }

    const finalUrl = await getFinalImageUrl(imageUrl);

    // Validate content type
    const response = await fetch(finalUrl, { method: 'HEAD' });
    const contentType = response.headers.get('content-type');

    if (response.status !== 200) {
      Toast.show({
        type: 'error',
        text1: 'Error',
        text2: `Failed to fetch image. HTTP status ${response.status}`,
      });
      return false;
    }

    if (!contentType?.startsWith('image/')) {
      Toast.show({
        type: 'error',
        text1: 'Error',
        text2: 'The provided URL does not point to a valid image.',
      });
      return false;
    }

    // Request permissions
    const { status } = await MediaLibrary.requestPermissionsAsync();
    if (status !== 'granted') {
      Toast.show({
        type: 'error',
        text1: 'Permission Required',
        text2: 'Please grant permission to save images.',
      });
      return false;
    }

    // Determine file extension from content type
    const typeMap: Record<string, string> = {
      'image/jpeg': 'jpg',
      'image/png': 'png',
      'image/gif': 'gif',
      'image/webp': 'webp',
    };
    const fileExtension = typeMap[contentType] || 'jpg';

    const timestamp = new Date().getTime();
    const finalFileName = fileName || `image_${timestamp}.${fileExtension}`;
    const fileUri = `${FileSystem.documentDirectory}${finalFileName}`;

    // Download
    const downloadResult = await FileSystem.downloadAsync(finalUrl, fileUri);
    if (downloadResult.status !== 200) {
      Toast.show({
        type: 'error',
        text1: 'Error',
        text2: `Download failed with status ${downloadResult.status}`,
      });
      return false;
    }

    // Verify file exists
    const fileInfo = await FileSystem.getInfoAsync(downloadResult.uri);
    if (!fileInfo.exists) {
      Toast.show({
        type: 'error',
        text1: 'Error',
        text2: 'Downloaded file does not exist.',
      });
      return false;
    }

    // Save to photo library
    const asset = await MediaLibrary.createAssetAsync(downloadResult.uri);

    // iOS: Add to "Downloads" album
    if (Platform.OS === 'ios') {
      try {
        const album = await MediaLibrary.getAlbumAsync('Downloads');
        if (album) {
          await MediaLibrary.addAssetsToAlbumAsync([asset], album, false);
        } else {
          await MediaLibrary.createAlbumAsync('Downloads', asset, false);
        }
      } catch {} // Ignore album creation errors
    }

    // Clean up temp file
    try {
      await FileSystem.deleteAsync(downloadResult.uri, { idempotent: true });
    } catch {}

    if (showAlerts) {
      Toast.show({
        type: 'success',
        text1: 'Image saved to your photo library!',
      });
    }

    return true;
  } catch (error) {
    const errorMessage = error instanceof Error ? error.message : 'Failed to save image.';
    if (showAlerts) {
      Toast.show({
        type: 'error',
        text1: 'Error',
        text2: errorMessage,
      });
    }
    return false;
  }
};
```

## Usage Example

```typescript
const handleDownload = async () => {
  const success = await downloadImage('https://example.com/image.jpg');
  if (success) {
    console.log('Image saved!');
  }
};
```

## Gotchas

- **Redirect handling**: URLs with redirects must use `fetch(..., { redirect: 'follow' })`.
- **Content type validation**: Verify MIME type starts with `image/` to prevent downloading non-images.
- **Permissions**: Always request MediaLibrary permissions before saving.
- **iOS albums**: Auto-creates "Downloads" album. Manual cleanup required if needed.
- **Temp file cleanup**: Always delete the temporary file after copying to photo library.
