---
name: file-sharing
category: utils
stack: [expo-print, expo-sharing, expo-file-system, react-native]
keywords: [pdf-generation, sharing, receipt, html-to-pdf, document]
source-files: [libs/utils/shareReceipt.ts, libs/utils/receiptHtml.ts]
---

# File Sharing

## Problem
You need to generate a PDF from HTML content and share it via the device's native share sheet.

## When to Use
- Building receipt/invoice generation and sharing
- Creating downloadable documents from dynamic data
- Supporting PDF export for in-app content
- Enabling device-native sharing (AirDrop, email, etc.)

## Implementation

### Dependencies
```json
{
  "expo-print": "^13.0.0",
  "expo-sharing": "^13.0.0",
  "expo-file-system": "^16.0.0"
}
```

### Code

**HTML Template** (`libs/utils/receiptHtml.ts`):

```typescript
export const generateSubscriptionReceiptHtml = (data: {
  reference: string;
  amount: string;
  currency: string;
  createdAt: string;
  planDisplayName: string;
  userEmail: string;
  startDate: string;
  endDate: string;
}) => {
  return `
    <html>
      <head>
        <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
        <style>
          body { font-family: Arial, sans-serif; margin: 20px; }
          .container { max-width: 600px; margin: 0 auto; }
          .header { text-align: center; margin-bottom: 30px; }
          .item { display: flex; justify-content: space-between; padding: 10px 0; border-bottom: 1px solid #eee; }
          .total { font-weight: bold; font-size: 18px; margin-top: 20px; }
        </style>
      </head>
      <body>
        <div class="container">
          <div class="header">
            <h1>Receipt</h1>
            <p>${data.reference}</p>
          </div>
          <div class="item">
            <span>Plan:</span>
            <span>${data.planDisplayName}</span>
          </div>
          <div class="item">
            <span>Email:</span>
            <span>${data.userEmail}</span>
          </div>
          <div class="item">
            <span>Date:</span>
            <span>${data.createdAt}</span>
          </div>
          <div class="item">
            <span>Start Date:</span>
            <span>${data.startDate}</span>
          </div>
          <div class="item">
            <span>End Date:</span>
            <span>${data.endDate}</span>
          </div>
          <div class="total item">
            <span>Total:</span>
            <span>${data.currency} ${data.amount}</span>
          </div>
        </div>
      </body>
    </html>
  `;
};
```

**Share Function** (`libs/utils/shareReceipt.ts`):

```typescript
import * as Print from 'expo-print';
import * as Sharing from 'expo-sharing';
import * as FileSystem from 'expo-file-system/legacy';
import { generateSubscriptionReceiptHtml } from './receiptHtml';

type ShareCallback = {
  onSuccess?: () => void;
  onError?: (error: Error) => void;
  onCancel?: () => void;
};

const saveAndSharePDF = async (
  htmlContent: string,
  fileName: string,
  callbacks?: ShareCallback
) => {
  try {
    // Generate PDF from HTML
    const { uri } = await Print.printToFileAsync({
      html: htmlContent,
      base64: false,
    });

    const newPath = `${FileSystem.documentDirectory}${fileName}.pdf`;

    // Move PDF to document directory
    await FileSystem.moveAsync({
      from: uri,
      to: newPath,
    });

    // Share via native sheet
    await Sharing.shareAsync(newPath, {
      mimeType: 'application/pdf',
      dialogTitle: `Share ${fileName}`,
      UTI: 'com.adobe.pdf',
    });

    callbacks?.onSuccess?.();
  } catch (error: any) {
    callbacks?.onError?.(error);
  }
};

export const downloadReceipt = async (
  receiptData: any,
  callbacks?: ShareCallback
) => {
  const htmlContent = generateSubscriptionReceiptHtml(receiptData);
  await saveAndSharePDF(
    htmlContent,
    `Receipt_${receiptData.reference}`,
    callbacks
  );
};
```

## Usage Example

```typescript
import { downloadReceipt } from 'libs/utils';

const handleShareReceipt = async () => {
  await downloadReceipt(
    {
      reference: 'TXN-001',
      amount: '29.99',
      currency: 'USD',
      createdAt: new Date().toISOString(),
      planDisplayName: 'Premium Plan',
      userEmail: 'user@example.com',
      startDate: '2024-01-01',
      endDate: '2024-02-01',
    },
    {
      onSuccess: () => console.log('Receipt shared!'),
      onError: (error) => console.error('Share failed:', error),
    }
  );
};
```

## Gotchas

- **File system cleanup**: PDF is saved to document directory. Consider periodic cleanup or store in temporary directory.
- **Sharing cancellation**: onCancel not fired by default. User dismissing share sheet may not trigger callbacks.
- **HTML encoding**: Special characters in HTML must be properly escaped.
- **Large PDFs**: Very large HTML documents may cause memory issues. Consider pagination for long receipts.
- **PDF UTI (iOS)**: Use `com.adobe.pdf` for reliable PDF sharing on iOS.
