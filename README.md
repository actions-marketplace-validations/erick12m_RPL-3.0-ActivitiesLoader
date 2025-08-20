# RPL 3.0 Activities Loader

A GitHub Action that automatically detects and uploads modified/created activities to the RPL 3.0. This action monitors changes in the `activities` directory and sends them to the RPL 3.0 Activities API.

## Overview

This actions do the following:

1. **Detects Changes**: Monitors the `activities` directory for any modified or created files
2. **Authenticates**: Logs into the RPL 3.0 platform using provided credentials (must have teacher permissions on the target course)
3. **Processes Activities**: Uploads activity and tests files to the API
4. **Handles Categories**: Creates or updates activity categories as needed

## Activity Structure

Activities should be organized in the following structure:

```
activities/
├── {course_id}/
│   ├── {activity_name_unit_test}/
│   │   ├── activity.json
│   │   ├── files_metadata
│   │   └── unit_tests.*
│   │   └── {extra_files}
│   └── {activity_name_io_test}/
│   │   ├── activity.json
│   │   ├── files_metadata
│   │   └── io_tests.json
│   │   └── {extra_files}
```

### Required Files

- **`activity.json`**: Contains activity metadata
  ```json
  {
    "category_name": "Basic Programming",
    "category_description": "Fundamental programming exercises for beginners",
    "name": "Hello World",
    "description": "A simple Hello World program to get started with Python",
    "language": "python",
    "points": 10,
    "active": true,
    "compilation_flags": ""
  }
  ```

- **`files_metadata`**: Defines the file permissions for students. Display can be: `read`, `read_write`, `hidden`. 
  ```json
  {
    "file.extension": {
      "display": "read_write"
    },
    "file2.extension": {
      "display": "read"
    },
    "file3.extension": {
      "display": "hidden"
    }
  }
  ```
  
### Optional Files

It can be a unit test suite or IO test for each activity.

- **`unit_tests.*`**: The test suite for the activity. The extension can currently be: `.py`, `rs`, `.c`, or `.go`.

- **`io_test.json`**: An IO test for the activity.
 ```json
 [
    {
      "name": "Test basic functionality",
      "test_in": "Hello World",
      "test_out": "Hello World"
    }
 ]
 ```

## Usage

Add this action to your workflow:

```yaml
name: Upload Activities to RPL 3.0

on:
  push:
    branches: [ main ]

jobs:
  upload-activities:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 2  # Required to detect changes

      - name: Upload activities to RPL 3.0
        uses: erick12m/RPL-3.0-ActivitiesLoader@v1
        with:
          rpl_username: ${{ secrets.RPL_USERNAME }}
          rpl_password: ${{ secrets.RPL_PASSWORD }}
          activities_dir: activities # The directory containing the activities in the repository
```

## Required Secrets

You must set up the following secrets in your repository:

### `RPL_USERNAME`
 Your RPL 3.0 platform username

### `RPL_PASSWORD`
Your RPL 3.0 platform password

### Setting up Secrets

1. Go to your repository on GitHub
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Add each secret with the appropriate name and value

## Troubleshooting

### Common Issues

1. **Authentication Failed**
   - Verify your `RPL_USERNAME` and `RPL_PASSWORD` secrets are correct
   - Ensure your account has the necessary permissions

2. **API Health Check Failed**
   - Verify the API endpoints are accessible
   - Check if there are network connectivity issues

3. **files_metadata Validation Failed**
   - Ensure the files_metadata file exists
   - Verify JSON syntax is correct
   - Check that all referenced files exist in the activity directory
   - Ensure display values are only: `read`, `read_write`, or `hidden`

4. **No Changes Detected**
   - Make sure you're using `fetch-depth: 2` in the checkout action
   - Verify changes are in the correct `activities` directory

5. **File Upload Failures**
   - Ensure activity files follow the required structure
   - Check that `activity.json` and `files_metadata` are properly formatted


---

## Development testing (DEVELOPMENT ONLY, FOR DEBUGGING PURPOSES)


> [!CAUTION]
> Note that this is only for testing of a local instance of RPL 3.0, not for normal usage. 
> If the url parameters point to the production APIs it WILL AFFECT your course and activities

You can test the action manually by running the following command:
```bash
./detect_changes.sh > changes.txt
cat changes.txt | RPL_USERNAME=your_username RPL_PASSWORD=your_password USERS_API_BASE_URL=http://localhost:8000/api/v3 ACTIVITIES_API_BASE_URL=http://localhost:8001/api/v3 ./process_activities_changes.sh
```

Ensure to use the correct credentials and API URLs for your RPL 3.0 local testing instance (the previous example uses the docker-compose setup)
