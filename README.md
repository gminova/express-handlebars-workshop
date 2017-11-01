**Author**: [@PiotrBerebecki](https://github.com/PiotrBerebecki)  

**Maintainer**: [@PiotrBerebecki](https://github.com/PiotrBerebecki)

# Express Handlebars.js code-along workshop

This is a workshop designed as a code along to learn about html templating using Handlebars.js.
The main objectives of the workshop are to:
- be introduced to the MVC (Model - View - Controller) model
- learn how to serve dynamic files using Handlebars.js templates
- learn how to use Handlebars views, layouts and partials
- be introduced to Handlebars helpers

## What is a template engine and what is Handlebars.js?
- A template engine is basically software that is designed to combine reusable text (i.e. templates) with data in order to generate HTML.
- Handlebars.js is one of the most popular JavaScript templating engines that builds on top of the [Mustache](https://github.com/janl/mustache.js) templating engine and has therefore compatible syntax with Mustache.
- Handlebars is designed so that it minimises writing lots of JavaScript logic inside the templates. Unlike other template engines such as Jade, it requires full HTML code and can have embedded placeholders that can "hold" data. For example:
```hbs
<h1>{{title}}</h1>
<p>{{description}}</p>
```
- Handlebars can be used for both client-side and server-side rendering.

## Using Handlebars with Express
Express has extensive support for template rendering and can load and leverage Handlebars.js with the full use of:
- **layouts** (i.e. base templates)
- **partials** (smaller templates within templates) and
- **helpers** (functions used in templates to manipulate data) to ultimately render **views**.

## Dive into coding

**Follow along with these steps to refactor an existing site, removing lots of duplication in the html and taking advantage of handlebars' helpers.**

### Set Up

1. Clone this repo
1. `cd task/`
1. Run `npm install` to install all the Node dependencies
1. Review the existing folder structure

  Notice that in order to serve individual fruit pages we had to create a number of static `.html` pages.

  <img src="./images-readme/folder-structure-fruits-html.png" width="200px" height="auto"/>

  Notice also that we have two `.html` files to handle errors.

  Handlebars will allow us to avoid duplication by creating template views. In fact Handlebars will allow us to delete all those `.html` files.

### Install handlebars

 1. In the tasks directory run the follwing code:

    ```bash
    npm i --save express-handlebars
    ```

### Add handlebars to your project

1. Require express-handlebars in app.js:

    ```js
    const exphbs = require('express-handlebars')
    ```

2. We now need to tell express the location of the ```views``` folder and the views engine that we want to use:
    ```js
    app.set('views', path.join(__dirname, 'views'));
    app.set('view engine', 'hbs');
    ```

3. We use ```app.engine``` to tell express to use the view engine (express-handlebars) to render hbs files. We are also configuring express-handlebars by giving the various directories that the engine will search for, and the default layout.
    ```js
    app.engine(
      'hbs',
      exphbs({
        extname: 'hbs',
        layoutsDir: path.join(__dirname, 'views', 'layouts'),
        partialsDir: path.join(__dirname, 'views', 'partials'),
        defaultLayout: 'main',
      })
    )
    ```
  4. We now need to make the relevant folders. Create a ```views``` directory within src. This is what you set in the app.js file as the directory to hold the .hbs files.
  5. In src/views create a ```layouts``` directory.
  6. In src/views create a ```partials``` directory.


### Loading up main page

1. Create a ```home.hbs``` in the views directory
1. Navigate to public/index.html. Copy all the contents to ```home.hbs```.
1. Delete public/index.html.
1. In src/views/layouts create a main.hbs file and insert the following code:
    ```hbs
    {{{ body }}} 
    ```
    Body will be the hbs file we are rendering. So when we go to the '/' route it will be ```home.hbs```.
    6. To start the server in development mode use the npm script from the package.json by typing this in your terminal:
    ```bash
    npm run start:watch
    ```

    Load up the server and go to localhost:3000 in your browser; expect to see a 404 error. We are now going to serve up the ```home.hbs``` file.

7. Create a ```home.js``` in src/controllers

8. In ```home.js``` add the following code:
    ```js
    exports.get = (req, res) => {
      res.render('home');
    };
    ```
    Here ```res.render``` renders a view and sends a rendered HTML string to the client.

9. In src/controllers/index.js require in ```home.js```.

10. Add the following line after files are required in:
    ``` js
    router.get('/', home.get);
    ```

## Split up the contents into partials
We are now going to create partials to simplify our website. Partials allow us to reuse snippets of HTML preventing duplication.
1. In src/views/partials create the following files:
    * htmlHead.hbs
    * header.hbs
    * footer.hbs
2. From home.hbs cut:
    * head to htmlHead.hbs
    * header to header.hbs
    * footer to footer.hbs
3. The next stage is to continue creating the ```main.hbs``` layout and inserting in the partials we have created. Copy across everything inside ```home.hbs``` except for the section tags and their content.
4. Place ```{{{body}}}``` inside the ```<main>``` tags.
5. Now you can insert the partials you have created into the ```main.hbs``` file. Use the [handlebars documentation](http://handlebarsjs.com/) to find out how.

## Creating The Fruits Page
1. In src/views create a ```fruits.hbs``` file
1. Open public/fruits.html and copy its contents to the ```fruits.hbs``` file
1. Delete ```fruits.html```
1. Delete everything from ```fruits.hbs``` that you have created partials for.

**See all the many list items? We will create these programmatically using handlebars.**

5. Delete all but one of the list items.

**Open up src/model/index.js and take a look. We will use this list for creating out fruits page.**

6. Now go back to src/controllers/fruits.js and require in the model of fruits.
    ```js
    const fruits = require('./../model/index');
    ```

    ```res.render``` also takes an object as an argument which can be used in the hbs file to insert content. Content will be available as a variable named as the object key(s).

7.Change the render function in ```fruits.js``` to the following:

    ```js
    exports.get = (req, res) => {
      res.render('fruits', { fruits });
    };
    ```
  We have passed the fruits model array to the ```render``` function, thereby making it available in ```fruits.hbs```.

## Using Helpers

**We can now use handlebars' built in helpers to dynamically generate HTML. We are going to use the ```each``` helper to iterate over the fruits array  (similarly to javascripts forEach array method).**

1. We want to iterate over the fruits array for each one creating a list item. 

    ```hbs

        {{#each fruits}}

        <li class="fruit__item">
          <a class="fruit__link" href="/fruits/{{this}}"><div class="fruit__image" style="background-image: url(/images/fruits/{{this}}.jpg)"></div></a>
          <div class="fruit__info">
            <h2 class="fruit__name">{{capitalize this}}</h2>
            <svg class="fruit__fav-btn"><use id="js-fav-btn" xlink:href="#icon-heart"></use></svg>
          </div>
        </li>

        {{/each}}

    ```
    * We are iterating over the fruits array creating a new ```<li>``` and all its content for each item.
    * The fruit array contains fruit names and the current value is inserted using the handlebars ```this``` keyword.



## Rendering Individual Fruits Pages

**In public/fruits we have multiple html files and we would like to create these all using a single hbs file. Appply the same steps as above to achieve this.**

Hints:

1. Copy across the relevant HTML code to an hbs file.
2. Change the controllers/singlefruit.js.
3. In the previous example we passed an array as an object property. You can make as many key value pairs on the object as you want, and they don't have to be arrays.

## Helpers
You have come across the ```each``` and ```if``` helpers that are built in with handlebars, but we can also create our own helpers in plain javascript.

1. Within views create a helpers directory
1. Within views/helpers create the following files:
* uppercase.js - this will contain a helper for making a string uppercase.
* index.js - this will be an index for all the helpers, and will export them on an object.

3. We want to to make all the letters in the fruit name in ```fruits.hbs``` uppercase. In ```uppercase.js``` add the following code:
    ```js
    module.exports = str => str.toUpperCase();
    ```
4. In ```index.js``` add the following code:
    ```js
    module.exports = {
      uppercase: require('./uppercase'),
    };
    ```
5. in src/app.js require in the helper from views/helpers/index.js:
    ```js
    const helpers = require('./views/helpers/index');
    ```
6. In the ```app.js``` update ```app.engine``` to pass the helpers object to express-handlebars.
    ```js
    app.engine(
      'hbs',
      exphbs({
        extname: 'hbs',
        layoutsDir: path.join(__dirname, 'views', 'layouts'),
        partialsDir: path.join(__dirname, 'views', 'partials'),
        defaultLayout: 'main',
        helpers,
      })
    );
    ```
7. We now need to use this in our ```singlefruit.hbs``` file. Update the ```<h1>``` element in the singlefruit.hbs as follows:
    ```hbs
      <h1 class="main__title">{{uppercase singleFruit}}</h1>
    ```

8. **Challenge:** Create a helper that capitalises the first letter of the fruit name title for each of the multiple fruits in ```fruits.hbs```. On the /fruits page the title associated with each of the multiple fruit images should be capitalised.


## (Stretch) Passing Objects to Change CSS 

**Challenge: Underline the current page link. When on the home page, home should be underlined, and when on the fruits page, fruits should be underlined.**

Hints
* Pass an object from within the relevant controller to help you change the css appropriately.
* Use the appropriate handlebars helper.

## (Extra) Error Refactor
**We specified the default layout was main.hbs. We can override this directly when we render a page. We will use a specific error layout when rendering an error page.**

1. Create an error.hbs file in the views folder.
    ```html
    <section class="error">
      <p class="error__desc">{{statusCode}}. <span class="error__msg">{{errorMessage}}</span></p>
      <img class="error__image" src="/images/error.png" alt="error">
    </section>
    ```
1. Add an ```error.hbs``` file into views/layouts with the following code:
    ```hbs
    <!DOCTYPE html>
    <html>
      {{>htmlHead}}
    <body>

      <main class="main">
      {{{body}}}
      </main>

    </body>
    </html>
    ```

3. In error.js update the code as follows:
    ```js
    const path = require('path');

    exports.client = (req, res) => {
      res.status(404).render('error', {
        layout: 'error',
        statusCode: 404,
        errorMessage: 'Page not found',
      });
    };

    exports.server = (err, req, res, next) => {
      res.status(500).render('error', {
        layout: 'error',
        statusCode: 500,
        errorMessage: 'Internal server error',
      });
    };
    ```
    Note that we specified the layout to be used by express handlebars.


## Resources
- [Handlebars Docs](http://handlebarsjs.com/)
- [express-handlebars docs](https://www.npmjs.com/package/express-handlebars)
- [Write Templates Like A Node.js Pro: Handlebars Tutorial](https://webapplog.com/handlebars/)
