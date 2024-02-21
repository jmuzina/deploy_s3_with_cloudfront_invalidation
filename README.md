# Deploy S3 With Cloudfront Invalidation
Deploys an artifact to Amazon S3, and creates a Cloudfront Invalidation to ensure the latest files are served.

## Inputs:
<table>
    <thead>
        <th>Name</th>
        <th>Description</th>
        <th>Type</th>
        <th>Default</th>
    </thead>
    <tbody>
        <tr>
            <td>environment</td>
            <td>Name of the environment you are deploying the artifact to</td>
            <td>string</td>
            <td>prod</td>
        </tr>
        <tr>
            <td>cloudfront_invalidation</td>
            <td>Path or pattern within your web artifact for which to create a Cloudfront invalidation</td>
            <td>string</td>
            <td>/*</td>
        </tr>
    </tbody>
</table>

## Secrets:
<table>
    <thead>
        <th>Name</th>
        <th>Description</th>
    </thead>
    <tbody>
        <tr>
            <td>AWS_S3_BUCKET_NAME</td>
            <td>Name of the S3 bucket to upload static web build artifacts to</td>
        </tr>
        <tr>
            <td>AWS_ACCESS_KEY_ID</td>
            <td>AWS IAM Access Key ID for accessing resources</td>
        </tr>
        <tr>
            <td>AWS_SECRET_ACCESS_KEY</td>
            <td>AWS IAM Secret Access Key for accessing resources</td>
        </tr>
        <tr>
            <td>AWS_CLOUDFRONT_DISTRIBUTION_ID</td>
            <td>ID of the Cloudfront Distribution used by this artifact</td>
        </tr>
        <tr>
            <td>AWS_REGION</td>
            <td>AWS deployment region. See <a href="https://www.cloudregions.io/aws/regions" target="_blank" referrerpolicy="no-referrer">AWS Docs</a> for list of valid values.</td>
        </tr>
    </tbody>
</table>


## Artifact name
Note: in your CI step before calling this workflow, you **must** upload your build artifact using 
[actions/upload-artifact](https://github.com/actions/upload-artifact) and assign the artifact name using the format
`${{ github.event.repository.name }}-${{ environment }}-${{ github.run_number }}`, where `environment` is some string
to distinguish runs of the same repository but different environment.

## Example:
```yaml
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    environment: dev
    steps:
      - name: Checkout Repo
        uses: actions/checkout@v2

      - name: Install JS dependencies
        run: npm i

      - name: Build artifact
        run: npm run build

      - name: Archive build artifact
        uses: actions/upload-artifact@v4
        with:
          # Artifact name must match naming convention ${{ github.event.repository.name }}-${{ environment }}-${{ github.run_number }}
          name: ${{ github.event.repository.name }}-dev-${{ github.run_number }}
          path: ./dist/your_project_name
  deploy:
    needs: build
    name: Deploy
    uses: 'jmuzina/deploy_s3_with_cloudfront_invalidation/.github/workflows/deploy_s3_cloudfront.yml@1.0.0'
    with:
      environment: dev
    secrets:
      # Make sure you store your secrets in repository actions secrets. Do not store them in cleartext for security reasons.
      AWS_S3_BUCKET_NAME: ${{ secrets.AWS_S3_BUCKET_NAME }}
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      AWS_CLOUDFRONT_DISTRIBUTION_ID: ${{ secrets.AWS_CLOUDFRONT_DISTRIBUTION_ID }}
      AWS_REGION: ${{ secrets.AWS_REGION }}
```