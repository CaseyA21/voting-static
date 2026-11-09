# Deploy the pages

The API does not serve the pages. They are files in `ui/`, uploaded to the website bucket.

In `ui/config.js`, replace `YOUR_DNS_NAME` with your hostname. Leave `http://` in place.

From the project root:

```bash
aws s3 sync ui/ s3://voting-static-<username> --region us-east-1 --delete
```

Replace `<username>` with the name you gave the bucket. Open the bucket website endpoint.

Run the same command after you change a page. It does not restart the API.
