I tried your `next.config.mjs` configuration as shown below, but I got the same error:
```js
const nextConfig = {
  reactStrictMode: true,
  distDir: 'build',       
  output: 'standalone',  
};

export default nextConfig;

```
To reslove it remove the following below lines from your YAML file:
 ```yml
 mv ./app ./build/standalone
 mv ./next ./build/standalone
```

Then, update your YAML file as shown below. 
>I referred to this [link](https://medium.com/@dileepa.mabulage/deploying-a-next-js-14-app-to-azure-app-service-using-github-actions-f5119a56e9f4) for deploying a Next.js 14 app to Azure App Service using GitHub Actions.

```yml


name: Build and deploy Node.js app to Azure Web App - ravi89sam89

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up Node.js version
        uses: actions/setup-node@v3
        with:
          node-version: '18.x'

      - name: npm install, build, and test
        run: |
          npm install
          npm run build --if-present
          npm run test --if-present
          mv ./public ./build/standalone/public

      - name: Zip artifact for deployment
        run: zip release.zip ./* -r

      - name: Upload artifact for deployment job
        uses: actions/upload-artifact@v3
        with:
          name: node-app
          path: release.zip

  deploy:
    runs-on: ubuntu-latest
    needs: build
    environment:
      name: 'Production'
      url: ${{ steps.deploy-to-webapp.outputs.webapp-url }}

    steps:
      - name: Download artifact from build job
        uses: actions/download-artifact@v3
        with:
          name: node-app

      - name: Unzip artifact for deployment
        run: unzip release.zip

      - name: 'Deploy to Azure Web App'
        id: deploy-to-webapp
        uses: azure/webapps-deploy@v2
        with:
          app-name: 'ravi89sam89'
          slot-name: 'Production'
          publish-profile: ${{ secrets.AZUREAPPSERVICE_PUBLISHPROFILE_4 }}
          package: './build/standalone'

```

Additionally, add the Startup Command to `node server.js` in the **Configuration**.There is no need to add/change physical path in Virtual path.

![enter image description here](https://i.imgur.com/nc0rPrm.png)

**Deployment status:**
![enter image description here](https://i.imgur.com/NtpRZYw.png)

**Output:**
![enter image description here](https://i.imgur.com/GQdqVK3.png)
