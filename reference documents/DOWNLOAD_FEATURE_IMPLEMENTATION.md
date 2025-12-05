# How We Implemented the 'Download All' Feature

**Last Updated:** 2023-11-25

This document provides a comprehensive, step-by-step log of how the 'Download All' media feature was implemented in the SafeCast project. It covers the initial failed attempts, the final successful architecture, and the critical security configurations required.

---

### 1. The Challenge: Downloading Multiple Files

The initial requirement was to allow operators to download all media files associated with an incident as a single zip archive. This presented a challenge, especially when dealing with large files like videos and high-resolution images.

### 2. Attempt #1 (Failed): Server-Side Zipping

Our first approach was to handle the zipping process on the backend. The plan was as follows:

1.  The frontend would send a list of S3 file URLs to a new `/api/download-media` endpoint.
2.  The backend server would fetch each file from S3, create a zip archive in memory, and then stream the final zip file back to the user.

**Why it Failed:**

This approach proved to be unstable and not scalable. The process of downloading and zipping large files was slow and consumed a significant amount of server memory and CPU. This led to two critical problems:

*   **Server Crashes:** For incidents with large videos or many files, the process would often crash the Node.js server with an `Error: queue closed` message.
*   **Poor User Experience:** If a user refreshed the page during the long zipping process, the connection would break, the server would still crash, and the download would fail.

### 3. Attempt #2 (Successful): Client-Side Zipping

Based on the failures of the server-side approach, we re-engineered the feature to offload the entire zipping process to the user's browser. This is a modern, efficient, and far more reliable solution.

**The Architecture:**

1.  **Install Libraries:** We first installed two essential libraries for the frontend:
    ```bash
    npm install jszip file-saver
    ```

2.  **Fetch & Zip in Browser:** We completely rewrote the `handleDownloadAll` function in `pages/IncidentDetails.tsx` to perform the following steps directly in the browser:
    *   Fetch all media files in parallel directly from their S3 URLs.
    *   Use `jszip` to add each downloaded file to a zip archive in the browser's memory.
    *   Provide a real-time progress indicator to the user.
    *   Use `file-saver` to trigger the download of the final, client-generated zip file.

**The Final Code (`pages/IncidentDetails.tsx`):**

```javascript
import JSZip from 'jszip';
import { saveAs } from 'file-saver';

// ... other imports

const [isDownloading, setIsDownloading] = useState(false);
const [downloadProgress, setDownloadProgress] = useState({ percent: 0, current: 0, total: 0 });

const handleDownloadAll = async () => {
    if (mediaUrls.length === 0) return;

    setIsDownloading(true);
    setDownloadProgress({ percent: 0, current: 0, total: mediaUrls.length });

    const zip = new JSZip();
    let filesDownloaded = 0;

    try {
        await Promise.all(mediaUrls.map(async (url) => {
            try {
                const response = await fetch(url);
                if (!response.ok) {
                    throw new Error(`Failed to fetch ${url}: ${response.statusText}`);
                }
                const blob = await response.blob();
                const filename = url.substring(url.lastIndexOf('/') + 1);
                zip.file(filename, blob);
            } catch (fetchError) {
                console.error(`Skipping file due to error: ${url}`, fetchError);
                zip.file(`download-error-${filesDownloaded}.txt`, `Could not download file from: ${url}\nError: ${fetchError.message}`);
            }
            filesDownloaded++;
            const percent = Math.round((filesDownloaded / mediaUrls.length) * 100);
            setDownloadProgress({ percent, current: filesDownloaded, total: mediaUrls.length });
        }));

        zip.generateAsync({ type: 'blob' }).then((content) => {
            saveAs(content, `incident-${id}-media.zip`);
            setIsDownloading(false);
        });

    } catch (err) {
        console.error('Download process failed:', err);
        alert('An unexpected error occurred during the zipping process.');
        setIsDownloading(false);
    }
};
```

### 4. The Final Hurdle: S3 CORS Policy

After implementing the client-side zipping, we encountered a final problem. The browser's developer console showed a `TypeError: Failed to fetch` error for every file. This was a classic **Cross-Origin Resource Sharing (CORS)** issue.

**The Problem:** For security reasons, web browsers block JavaScript code from making requests to a different domain (in this case, our S3 bucket) unless that domain explicitly gives permission.

**The Solution:**

We had to add a CORS policy to our `safecast-media-uploads` S3 bucket to allow `GET` requests from our web application. This was done by adding the following JSON configuration in the AWS S3 console under **Bucket > Permissions > Cross-origin resource sharing (CORS)**:

```json
[
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "GET"
        ],
        "AllowedOrigins": [
            "https://3000-firebase-safecast-1763946605162.cluster-r7kbxfo3fnev2vskbkhhphetq6.cloudworkstations.dev",
            "http://localhost:3000"
        ],
        "ExposeHeaders": []
    }
]
```

This policy explicitly tells S3 that it is safe to serve files to our web application's domain.

---

### Conclusion

By moving the zipping process from the server to the client and by configuring the S3 bucket's CORS policy correctly, we successfully implemented a fast, reliable, and scalable "Download All" feature. This approach improves server stability and provides a much better experience for the operator.
