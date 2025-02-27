# Sharing a file anonymously on a OneDrive personal account fails

I am developing a .NET MAUI app that lets people collaborate with others using their cloud storage account and have run into the problem that anonymously sharing a file on OneDrive personal stopped working. Since recently, the `shareId` retrieved via the OneDrive REST API can only be used from the same account that shared the item.

Below is a set of repro steps that I hope can help resolve the issue :-)

Best regards
Lars Hammer, lars.hammer@dbquity.com

## Step 1. Create a file to share
On a OneDrive personal account for a.test@dbquity.com that I use for test purposes, I have created a folder, “Test”, that contains a single file, “ToShare.txt”:
![image](https://github.com/user-attachments/assets/703f0f40-7d4b-4ed0-a011-dd03143b8710)

## Step 2. Use the OneDrive REST API to obtain a `shareId` for the file
Next, I use Graph Explorer, https://developer.microsoft.com/en-us/graph/graph-explorer, to share that file anonymously as described here: https://learn.microsoft.com/en-us/onedrive/developer/rest-api/api/driveitem_createlink?view=odsp-graph-online using
```
POST https://graph.microsoft.com/v1.0/me/drive/root:/Test/ToShare.txt:/createLink
{ "type":"view", "scope":"anonymous" }
```
![image](https://github.com/user-attachments/assets/e8a9a1fe-0782-4dae-9916-35df63cb6ed8)
which gets me this response
```
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#microsoft.graph.permission",
    "id": "d12fa53e-f7bf-4c7a-87e2-5c5d148f16c1",
    "roles": [
        "read"
    ],
    "shareId": "u!aHR0cHM6Ly8xZHJ2Lm1zL3QvYy9kYjMxYzVhZmYwZDIyNzM3L0VVcVN6TndTVmRaQXFYRnpfMlNNOFE0QlZoSVBHVTdOclBEZGhzNmp6R05rX0E",
    "hasPassword": false,
    "link": {
        "scope": "anonymous",
        "type": "view",
        "webUrl": "https://1drv.ms/t/c/db31c5aff0d22737/EUqSzNwSVdZAqXFz_2SM8Q4BVhIPGU7NrPDdhs6jzGNk_A",
        "preventsDownload": false
    }
}
```

## Step 2a. Validate the `shareId`
While still logged in to the test account, I can access the shared item as documented here: https://learn.microsoft.com/en-us/onedrive/developer/rest-api/api/shares_get?view=odsp-graph-online using 
```
GET https://graph.microsoft.com/v1.0/shares/u!aHR0cHM6Ly8xZHJ2Lm1zL3QvYy9kYjMxYzVhZmYwZDIyNzM3L0VVcVN6TndTVmRaQXFYRnpfMlNNOFE0QlZoSVBHVTdOclBEZGhzNmp6R05rX0E/driveItem
```
![image](https://github.com/user-attachments/assets/b42e7c9d-fb4d-45d3-81e7-a2f3ee254377)

## Step 3. (try to) Access the shared file without being logged in to OneDrive
But when I log out of the test account and attempt to anonymously access the shared item using the same request it fails with this error:
```
{
    "error": {
        "code": "accessDenied",
        "message": "The sharing link no longer exists, or you do not have permission to access it.",
        "onedrive.linkFeatures@odata.type": "#Collection(oneDrive.linkFeatures)",
        "@onedrive.linkFeatures": []
    }
}
```
![image](https://github.com/user-attachments/assets/f05b6ed2-569f-4118-944f-c4ea0e81f823)

Previously, this anonymous access (i.e. without a bearer access token) to the share did work.

Access is also denied if I try to access the shared item when logged in to another OneDrive account.
