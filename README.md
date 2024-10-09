# IDE Settings

## Description
This repository stores configuration files for different development environments, langauges and IDEs. Each directory represents an IDE and subfolders represent the langauges, in case of frameworks or will be inside langauges folders.It contains the configuration files required for that environment.

## Repository Structure
- Visual Studio Code:
  - Python
  - React
- JetBrains Rider:
  - C#

### Ide settings
For files `launch.json` and `settings.json` you have to copy to `./.vscode` folder
And for `.gitignore` you have to copy to root project folder

### Extentions
Download the extentions.txt file to root project folder and run next script

    Get-Content .extensions.txt | ForEach-Object {
        code --install-extension $_
    }