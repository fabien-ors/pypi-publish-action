# pypi-publish-action
Simple github action to publish Python packages in PyPi repository using twine. This action assumes that all the Python packages (source or binary) have been uploaded through upload-artifact v4 action.

I made this following this guide :
https://docs.github.com/en/actions/creating-actions/creating-a-composite-action

**Important**:
- Only support **Linux** Github runner. But if you want to publish **Windows** or **Mac** R binary packages, the idea is to build under **Windows**/**Mac** and publish under **Linux** using this action (see usage).
- Artifacts are uploaded/downloaded using v4 version of upload-artifact and download-artifact Github actions

## Requirements
- The repository (either 'pypi' or 'testpypi')
- Your username for the given repository
- Your password for the given repository

## Input variables
* ```repo``` - Repository type ('pypi' or 'testpypi')
* ```username``` - Username for the repository
* ```password:``` - Password for the repository
* ```pattern``` - File name pattern filter (default = "*")

## Usage
Build Python packages under Windows and publish them to PyPi server


    name: build and publish Python packages (windows)
    on:
      workflow_dispatch:
        inputs:
          logLevel:
            description: 'Manual'
   
    jobs:
      
      build:
        runs-on: windows-2019
        strategy :
          matrix:
            python_version: ["3.11","3.12"]
            
        steps:
        - uses: actions/checkout@v3
    
        - name: Setup Python Version
          uses: actions/setup-python@v3
          with:
            python-version: ${{matrix.python_version}}
            
        - name : Create Python Package and store filename in github environment
          run : |
            python setup.py bdist_wheel
            $PKG_PATH = Get-ChildItem -Path "dist/*" -File
            echo "MY_PKG=$PKG_PATH" | Out-File -FilePath $Env:GITHUB_ENV -Encoding utf8 -Append
    
        - uses: actions/upload-artifact@v3
          # Use specific artifact identifier for publishing all Python versions
          with:
            name: windows-package-${{matrix.python_version}}
            path: ${{ env.MY_PKG }}
            
        
      publish:
        needs: build
        # Only ubuntu can upload via ssh
        runs-on: ubuntu-latest
        
        steps:
        - uses: fabien-ors/pypi-publish-action@v1
          with:
            repo: pypi
            username: ${{ secrets.PYPI_USERNAME }}
            password: ${{ secrets.PYPI_PWD }}


