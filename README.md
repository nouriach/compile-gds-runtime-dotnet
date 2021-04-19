## How to Import the GOV UK Design System into a .NET MVC Project

_The walkthrough will also detail how to set-up your project to compile the sass at runtime_

**author**: Nathan Ouriach

**peer review**: Rob Marshall

___

### Installing the correct npm packages

- Firstly, ensure you have [Node](https://nodejs.org/en/) installed 
- Open up your terminal and navigate to the root of your project folder
- Once inside the root of your project folder run the command `npm init`
- Then install the correct dependencies by running the following commands
```sh
  npm install sass
  npm install node-sass
  npm install govuk-frontend --save
  ```
    
- Inside your Visual Studio project, open up the Package Manager Console and install the following package: `Install-Package LigerShark.WebOptimizer.Core`

_[LigerShark.WebOptimizer.Core](https://libraries.io/nuget/LigerShark.WebOptimizer.Core) allows for the bundling and minification of CSS and JavaScript files at runtime._
___
### Updating your project

- Inside your .NET MVC project find the _wwwroot_ folder, and then inside that create a new folder called _scss_ 
- Then inside the new _scss_ folder create a `main.scss` file
- Replace everything in the `main.scss` file with : `@import "node_modules/govuk-frontend/govuk/all";`
- Now inside your _wwwroot_ folder there should be a _css_ folder, create a new css file called _main.css_ inside your `_Layout.cshtml` file you will need to add the following code to the bottom of your `<head>` element

```sh
  <head>
    //    
    <link rel-"stylesheet" href="~/main.css" />
  </head>
 ```
- Inside _wwwroot_ create a new folder and call it _assets_
- Inside your project's _node_modules_ (initiated when you ran `npm init` at the beginning) navigate to the following 
-- `/node_modules/govuk-frontend/govuk/assets/images`
-- `/node_modules/govuk-frontend/govuk/assets/fonts`
- Copy these to folders to your clipboard and paste them into your new _wwwroot/assets_ folder
___
### Setting up the Javascript

- Inside your `_Layout.cshtml` file  paste the following code into the top of your `<body>` element
```sh
  <body>
    <script>document.body.className = ((document.body.className) ? document.body.className + ' js-enabled' : 'js-enabled');
    </script>
    //
  </body>
``` 
          
- Inside your _wwwroot/js_ folder create a new file and call it `govuk.js`
- Return to your _node_modules_ folder and navigate to the following file: `/node-modules/govuk-frontend/govuk/all.js`
- Copy all of the content from this file and paste it into your new `govuk.js` file
- Now import thi sfile before the close `</body>` element inside your `Layout.cshtml` file and then run the `initAll()` function t0 initialise all the components.

```sh
  <body>
      <script>document.body.className = ((document.body.className) ? document.body.className + ' js-enabled' : 'js-enabled');
      </script>
        //
        //
      <script src="<wwwroot/js/govuk.js"></script>
      <script>
        window.GOVUKFrontend.initAll()
      </script>
  </body>
```
___
### Compile the .sass file

- Locate your `package.json` file and add the following script
```sh
"scripts" : {
    "compile-sass" : "node-sass ~[full path from root of your project]/wwwroot/scss/main.scss [full path from root of your project]/wwwroot/css/main.css"
 } 
```
- This tells the compiler to target a specific sass file and render it into a specific css file
- In the command line rune - and in the same location as your `package.json` file - run the command: `npm run compile-sass`
- To test it has worked, select a random component from the [GOV UK Design System](https://design-system.service.gov.uk/components/) and paste the HTML into one of your `.cshtml` View files
- Run the project and then you should see the chosen component rendered correctly
___
### How to compile the sass at runtime using gulp
- Install the following dependencies:
```sh
    "gulp": "^4.0.2",
    "gulp-clean-css": "^4.3.0",
    "gulp-dart-sass": "^1.0.2",
    "gulp-rename": "^2.0.0",
    "gulp-uglify": "^3.0.2",
```
- This can be done by running the below command
```sh
npm install gulp
```

- In the root of your folder create a new file called `gulp.js` (this will exist at the same level as your `Startup.cs` file)
- At the top of your new `gulp.js` file import the above dependencies, your code will look like the below

```sh
var sass = require("gulp-dart-sass");
var async = require("async");
var rename = require("gulp-rename");
var cleanCSS = require('gulp-clean-css');
var uglify = require('gulp-uglify');
// //
```

- You will then create a function underneath these imports that will build the sass by grabbing all of your projects `.scss` files  and directing them to your new `main.css` file. Your function will look like the below:
```sh
const buildSass = () => gulp.src("wwwroot/css/*.scss")
	.pipe(sass().on("error", sass.logError))
	.pipe(cleanCSS())
	.pipe(gulp.dest("wwwroot/css"));
```
You will also need a function to copy over the assets from the _node_modules_ folder and place them within your _wwwroot/assets_ folder, as well as bringing over the `.js` file contents into your project's `govuk.js` file:
```sh
const copyGovukAssets = () => gulp.src(["node_modules/govuk-frontend/govuk/assets/**/*"]).pipe(gulp.dest("wwwroot/assets")).on("end", () =>
	    gulp.src(["node_modules/govuk-frontend/govuk/all.js"]).pipe(rename("govuk.js")).pipe(gulp.dest("wwwroot/js/")));
	    gulp.src(["node_modules/govuk-frontend/govuk/all.js"]).pipe(uglify()).pipe(rename("govuk.js")).pipe(gulp.dest("wwwroot/js/")));
```
Close your `gulp.js` file by creating a task that runs the above functions
```sh
gulp.task("build-fe", () => {
	return async.series([
		(next) => buildSass().on("end", next),
		(next) => copyGovukAssets().on("end", next)
		(next) => copyGovukAssets().on("end", next),
	])
}); 
```
___
### Add the gulp tasks to the `package.json` file to run at compile time
- Locate your `package.json` file and add the following script 
- By adding the below script you will be able to run the gulp task during the deployment process
```sh
{
    //
    "scripts": {
            "build": "gulp build-fe"
    }
    //
}
```
___
### If using GitHub Actions, setup a new workflow 
- Open your repo in GitHub and select the _Actions_ tab
- Follow the prompts to set up a new .NET GitHub workflow
- Rename the file to whatver you want
- Ensure that the workflow runs the script within `packagee.json` (earlier on we stored the gulp task under _build_)
```sh
name: .NET

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Setup .NET
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 5.0.x
    - name: Restore dependencies
      run: dotnet restore
    - name: Build
      run: dotnet build --no-restore
    - name: Test
      run: dotnet test --no-build --verbosity normal
```
- In the code above the below steps will call the gulp task from within `packagee.json` and the steps will be run every time a commit is pushed to `master`
```sh
    - name: Build
      run: dotnet build --no-restore
  ```
