# Generate Big Files Using GitHub Actions

This GitHub Action allows you to create a file with random text of a specified size.

## How to Use

1. **Trigger the Workflow Manually:**

   Go to the "Actions" tab in your GitHub repository. Find the workflow named "Generate Big Files Using GitHub Actions" and click on it. Click the "Run workflow" button.

2. **Specify the File Size and Unit:**

   When you trigger the workflow manually, you will be prompted to enter the size of the file and the unit. You can specify the size (e.g., 200) and the unit (B, KB, MB, GB). The default value is 1B. Note that the maximum file size allowed is 1GB.

3. **Run the Workflow:**

   After entering the desired file size and unit, click the "Run workflow" button again to start the process.

## Example

To create a file of 1GB:

1. Go to the "Actions" tab.
2. Select "Generate Big Files Using GitHub Actions" from the list of workflows.
3. Click on "Run workflow."
4. Enter `1` in the input field for `file_size`.
5. Enter `GB` in the input field for `size_type`.
6. Click "Run workflow."

The action will create a file named `1GB.md` with random text of the specified size and commit it to your repository using the name and email of the most recent commit author.

## Workflow File

Below is the GitHub Action workflow file:

```yaml
name: Generate Big Files Using GitHub Actions

on:
  workflow_dispatch:
    inputs:
      file_size:
        description: 'Size of the file (e.g., 200)'
        required: true
        default: '1'
      size_type:
        description: 'Unit of the file size (B, KB, MB, GB)'
        required: true
        default: 'B'

jobs:
  create-file:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Get git user name and email
      id: git-info
      run: |
        GIT_NAME=$(git log -1 --pretty=format:'%an')
        GIT_EMAIL=$(git log -1 --pretty=format:'%ae')
        echo "GIT_NAME=$GIT_NAME" >> $GITHUB_ENV
        echo "GIT_EMAIL=$GIT_EMAIL" >> $GITHUB_ENV

    - name: Create file with specified size
      run: |
        FILE_SIZE=${{ github.event.inputs.file_size }}
        SIZE_TYPE=${{ github.event.inputs.size_type }}
        case $SIZE_TYPE in
          B|b) MULTIPLIER=1 ;;
          KB|kb|Kb|kB) MULTIPLIER=1000 ;;
          MB|mb|Mb|mB) MULTIPLIER=1000000 ;;
          GB|gb|Gb|gB) MULTIPLIER=1000000000 ;;
          *) echo "Invalid unit"; exit 1 ;;
        esac
        FILE_SIZE_BYTES=$(( FILE_SIZE * MULTIPLIER ))

        if [ $FILE_SIZE_BYTES -gt 1000000000 ]; then
          echo "File size cannot be greater than 1GB"; exit 1;
        fi

        FILE_NAME="${FILE_SIZE}${SIZE_TYPE}.md"
        echo "Creating file: $FILE_NAME with size: $FILE_SIZE_BYTES bytes"
        base64 /dev/urandom | head -c $FILE_SIZE_BYTES > "$FILE_NAME"
        ls -lh "$FILE_NAME"

    - name: Commit and push file
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      run: |
        FILE_NAME="${{ github.event.inputs.file_size }}${{ github.event.inputs.size_type }}.md"
        echo "Adding file: $FILE_NAME"
        git config --global user.name "${{ env.GIT_NAME }}"
        git config --global user.email "${{ env.GIT_EMAIL }}"
        git add "$FILE_NAME"
        git commit -m "Add random text file of size ${FILE_SIZE} ${SIZE_TYPE}"
        git push https://x-access-token:${{ secrets.GITHUB_TOKEN }}@github.com/${{ github.repository }}.git HEAD:main
