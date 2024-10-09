## Ide settings
For files `launch.json` and `settings.json` you have to copy to `./.vscode` folder
And for `.gitignore` you have to copy to root project folder

## Extentions
Download the `extentions.txt` file to root project folder and run next script

    Get-Content .extensions.txt | ForEach-Object {
        code --install-extension $_
    }

## Pylint settings
Download the `.pylintrc` file to root project folder and run pylint in terminal