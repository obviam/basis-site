# basis-site
Basis-site is a static site generator. It takes an input directory of files, (optionally) transforms them, and copies the results to an output directory which you can serve as a set of static files using the web server of your choice.

 It is build on top of [basis-arguments](https://github.com/badlogic/basis-arguments) for CLI argument parsing, and [basis-template](https://github.com/badlogic/basis-template) for templating.

 Basis-site is geared towards users with programming experience due to the heavy use of basis-template's templating scripting language.

# Motivation
Why another static site generator?

* Tries not to do and be everything to everyone. Basis-site comes only with a handful of simple rules to apply to your static site generation. Bring your own site structure and functionality to be consumed by your templates.
* Can be integrated in a JVM web app that serves dynamic content, e.g. comments on a blog.
* Can be easily extended with any JVM language. Want to minify your static CSS/JS/HTML?
* Uses a more powerful templating language than Hugo and consorts.
* Only depends on basis-argument and basis-template, both having zero dependencies themselves.

# Usage
Basis-site can be either used from the command line, or as a dependency of your JVM project.

# Command line usage

## Setup
You can run the basis-site `.jar` file as an app without needing a JVM code project. All you need is an installation of Java 8+ available in your `$PATH`. You can download the latest (snapshot) version of the `.jar` from [here](https://libgdx.badlogicgames.com/ci/basis-site/basis-site.jar).

You can also build the far `.jar` from source via Maven:

```
mvn clean package
```

The `basis-site.jar` will end up in the `target/` folder.

With Java and the `.jar` you can now start basis-site on the command line like this:

```bash
java -jar basis-site <arguments>
```

## A basic site
Basis-site takes the files in an input folder, (optionally) transforms them, and writes the result to an output folder.

> Note: Basis-site relies heavily on [basis-template](https://github.com/badlogic/basis-template). Before continuing, it's highly recommended to please read basis-template's [documentation](https://github.com/badlogic/basis-template#basis-template). You can ignore the Java parts of basis-template. Basis-site takes care of that!

Let's assume we want to generate a static site consisting of two pages, a landing page, and an about page. Both should share the same header and footer. Our input folder could look like this:

```
input/
    _templates/
        header.html
        footer.html
    css/
        style.css
    js/
        code.js
    index.bt.html
    about.bt.hml
```

The index and about page files contain the infix `.bt` in their file names. This signals basis-site that these files should be transformed by evaluating them as [basis-template](https://github.com/badlogic/basis-template) templates, and strip the `.bt.` infix from the output file name (`index.html` instead of `input.bt.html`). We can use basis-template `include` statements in each file to include the header and footer:

```html
<!-- index.bt.html -->
{{include "_templates/header.html"}}

<h1>Welcome to my website</h1>

<p>You can learn more about me on the <a href="about.html">About page</a></p>

{{include "_templates/footer.html"}}
```

```html
<!-- about.bt.html -->
{{include "_templates/header.html"}}

<h1>About me</h1>

<p>I'm a little pea, I love the birds and the trees. Go back to the <a href="index.html">landing page</a></p>

{{include "_templates/footer.html"}}
```

Note how we link to `index.html` and `about.html` instead of `index.bt.html` and `about.bt.html`. This is because the output names of these two files will be stripped of the `.bt.` infix after they've been evaluated as templated files!

The include paths are specified relative the the file the other files are included in.

The header and footer files look like this:

```html
<!-- header.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <link href="/css/style.css" rel="stylesheet">
    <script src="/js/code.js"></script>
</head>
<body>
   <!-- Imagine the markup for a navbar here -->
```

```html
<!-- footer.html -->
</body>
</html>
```

Let's generate the static output files from our input folder via the command line:

```bash
$ java -jar basis-site.jar -d -i input/ -o output/
```

This will delete the output folder (`-d`), take the files in the input folder (`-i`), transform them, and copy them to the output folder (`-o`). The output folder will look like this:

```
output/
    css/
        style.css
    js/
        code.js
    index.html
    about.html
```

Basis-site did not copy the files in `_templates/` because the name of the folder they are contained in starts with an underscore `_`. In general, basis-site will not transform or copy any files and folders (and their sub-folders) starting with an underscore. This lets us structure our input folder however we want it, while keeping the output clean.

For the two input files `input.bt.html` and `about.bt.html`, basis-site ran their contents through the basis-template templating engine, and wrote the results to the output files `index.html` and `about.html`, stripping the `.bt.` infix from the file names.

The `style.css` and `code.js` files were copied verbatim, retaining the folder structure they were found in.

**Key take-aways**

1. Files and folders with an underscore `_` at the start of their name are ignored.
2. Files containing the `.bt.` infix in their are run through the basis-template templating engine. The resulting content is written to files with the `.bt.` infixed stripped from their name.
3. All other files and folders are copied verbatim.

## Watch mode
Having to invoke the basis-site command line app after every change of our site gets old fast. Basis-site thus lets you start it in watch mode with the `-w` flag.

```bash
$ java -jar basis-site -d -w -i input -o output
```

```
00:00  INFO: Watching input directory input
01:54  INFO: Deleting output directory output.
01:54  INFO: Processed input/css/style.css -> output/css/style.css
01:54  INFO: Processed input/js/code.js -> output/js/code.js
01:54  INFO: Processed input/blog/hello-world/index.html -> output/blog/hello-world/index.html
01:54  INFO: Processed input/blog/another-post/index.html -> output/blog/another-post/index.html
01:54 ERROR: Error (input/about.bt.html:3): Expected ':', but got '='

        title = "Ponyhof - About"
              ^
```

In watch mode, basis-site will re-generate the site if a file or folder in the input directory was changed (created, modified, deleted, renamed). You can stop the app by pressing `CTRL+C`.

## Metadata
Let's be good web citizens and set the `<title>` of each page, e.g. `Ponyhof` for the landing page, and `Ponyhof - About` for the about page.

The `<title>` tag is a child of the `<header>` tag, which is located in `_templates/header.html`. This header file is included in both the landing and about page. How do we inject page specific values?

The answer is basis-template. Let's add some metadata to our landing and about pages:


```html
<!-- index.bt.html -->
{{ metadata = { title: "Ponyhof" } }}
{{include "_templates/header.html"}}

<h1>Welcome to my website</h1>

<p>You can learn more about me on the <a href="about.html">About page</a></p>

{{include "_templates/footer.html"}}
```

```html
<!-- about.bt.html -->
{{ metadata = { title: "Ponyhof - About" } }}
{{include "_templates/header.html"}}

<h1>About me</h1>

<p>I'm a little pea, I love the birds and the trees. Go back to the <a href="index.html">landing page</a></p>

{{include "_templates/footer.html"}}
```

Since we define `metadata` before including the header in each file, its contents will be available to any templating code inside `header.html`.

We can now modify `header.html` to use the metadata:

```html
<!-- header.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>{{metadata.title}}</title>
    <link href="/css/style.css" rel="stylesheet">
    <script src="/js/code.js"></script>
</head>
<body>
   <!-- Imagine the markup for a navbar here -->
```

> Note: `header.html` is evaluated as a template, even though it misses the `.bt.` infix. This is because it is included in files that basis-site evaluates as templates due to the `.bt.` infix in their names.

When we regenerate our static content on the command line, the resulting `index.html` and `about.html` files will each have a `<title>` tag in them with their respective `metadata.title` value injected!

Using a basis-template [map literal](https://github.com/badlogic/basis-template#literals) to specify the title of each page may seem overkill for this example. But we can add other metadata in the same `metadata` map, like tags or publication date, which we can then refer to within the templated file.

Having a metadata block at the top of our templated file is a nice way to define file specific, well, metadata in one place and reuse that data throughout the file (or includes files as in the case of `header.html`).

Of course you could choose a variable name other than `metadata`, or place the code span defining the metadata at a location other than the top of the file. But if you specify the metadata like in the example above, basis-site can help you do more powerful things!

**Key take-aways**
1. You can pass information from a templated file to its included files by defining a variable in the including file (e.g. `index.bt.html`), and referencing that variable in the included file (e.g. `header.html`).
2. Specify metadata for a file such as title, publication date, or tags, in a basis-template code span at the top of file, defining a variable called `metadata` using a map literal.

## Built-in functions
Let's make our site a blog! We want our blog posts to have URLs like `https://mysite.com/blog/url-of-the-post/`. Since we are serving static files, and assuming our web server will serve a file called `index.html` when no file is specified in the URL, our folder structure should look likes this:

```
input/
    _templates/
        header.html
        footer.html
    css/
        style.css
    js/
        code.js
    blog/
        hello-world/
            index.bt.html
            a-nice-image.jpg
        another-post/
            index.bt.html
    index.bt.html
    about.bt.hml
```

Our "Hello world" `index.html` could look like this:

```html
<!-- blog/hello-world/index.html -->
{{
    metadata = {
        title: "Hello world",
        published: true,
        date: "2018/06/23 21:00"
    }
}}
{{include "../../_templates/header.html"}}

<h1>{{metadata.title}}</h1>
<span>{{metadata.date}}</span>

<p>This is my first post!</p>

<img src="a-nice-image.jpg">

{{include "../../_templates/footer.html"}}
```

We specified `metadata` that gets used in both the `header.html` file and within the post itself.

The "My second post" file would have the same overall structure, but with different metadata and content. It's metadata could look like this:

```html
<!-- blog/another-post/index.html -->
{{
    metadata = {
        title: "Another post",
        published: false,
        date: "2018/07/03 20:15"
    }
}}
... includes and content ...
```

Running basis-site, we'll get `output/blog/hello-world/index.html` and `output/blog/another-post/index.html` as we'd expect. It will also copy the image `a-nice-image.jpg`.

Nobody will ever know about our blog posts, unless we link them from the landing page. How do we get a list of blog posts into the landing page?

Basis-site provides a handful of functions to every template it evaluates, one of which is `listFiles(String path, boolean withMetadataOnly, boolean recursive)`.

This function will return the files in the specified path, which is relative to the input directory. E.g. `"blog/"` would return the files in the `input/blog/` directory. If we pass `true` for `withMetadataOnly`, then only files that define a `metadata` map in their first basis-template code span will be returned (like our blog post files above). Finally, if we pass `true` for `recursive`, the function will not only return the files in the specified folder, but also all files in its sub-folders.

We can use this function to iterate through all our blog post files (and their metadata) within our landing page:

```html
<!-- index.bt.html -->
{{ metadata = { title: "Ponyhof" } }}
{{include "_templates/header.html"}}

<h1>Welcome to my website</h1>

<p>You can learn more about me on the <a href="about.html">About page</a></p>

<h2>Blog posts</h2>
<ul>
{{for file in listFiles("blog/", true, true)}}
    {{if file.metadata.published == false continue end}}
    <li><a href="{{file.getUrl()}}">{{file.metadata.date}} - {{file.metadata.title}}</a></li>
{{end}}
</ul>

{{include "_templates/footer.html"}}
```

The call to `listFiles()` returns a `List<SiteFile>` of all the files it found (recursively) in the `blog/` folder that have a `metadata` definition in their first basis-template code span.

Instances of `SiteFile` have a field `metadata` which contains the contents of the `metadata` map as specified in the template code of the file.

We can then iterate through all these files, and for each `published` file, we output a `<li>` containing a link to the blog post, displaying the blog posts publication date and title. We use the `SiteFile#getUrl()` function to get a URL for the folder the file is contained in (e.g. `blog/hello-world/` for the `blog/hello-world/index.bt.html` file).

By specifying metadata and adding 5 lines of template code to our landing page, we now have a fully functioning blog! Well, almost.

The order in which `listFiles()` returns the files is undefined. Basis-site provides the function `sortFiles(List<SiteFile> files, String fieldName, boolean ascending)` to all template files that allows them to sort the files by one of the metadata fields in ascending or descending order. This only works on fields that are `Comparable`, like numbers, strings or dates.

```html
{{for file in sortFiles(listFiles("blog/", true, true), "date", false)}}
   ...
{{end}}
```

This sorts the listed files by the metadata field `date`in descending order. Great!

But there's a problem: the `date` of each blog post is currently a string. We can change this to a `Date` by using the `parseDate(String date)` function. It expects a string of the format `yyyy/mm/dd hh:ss`, e.g. `2018/07/03 21:32`. We can change the `date` metadata in our blog posts as follows:

```html
{{
    metadata = {
        ...
        date: parseDate("2018/07/03"),
        ...
    }
}}
```

Now the `sortFiles()` function will sort by proper date, not lexicographically by string!

To finish off our new blog, we have to fix the formatting of the post dates when we display them on the landing page and the post pages. For that we can use the `formatDate(String dateFormat, Date date)` function:

```html
{{formatDate("yyyy/MM/dd", file.metadata.date)}}
```

The date format string follows the syntax of Java's [SimpleDateFormat](https://docs.oracle.com/javase/6/docs/api/java/text/SimpleDateFormat.html).

**Key take-aways**
1. Use the `listFiles()` functions to get a list of files with their metadata.
2. Use the `sortFiles()` function to sort a list of files by a field in their metadata.
3. Use the `parseDate()` and `formatDate()` functions to convert strings to `Date` instances and vice versa.

## Examples
You can find the final result of the above tutorial in the [`example/`](example/) folder.

# Embedding and extending basis-site
While command line usage of basis-site is likely sufficient for simple sites, you can of course also embed and extend it in code form in your JVM app.

## Setup
As a dependency of your Maven project:

```
<dependency>
   <groupId>io.marioslab.basis</groupId>
   <artifactId>site</artifactId>
   <version>1.4</version>
</dependency>
```

As a dependency of your Gradle project:
```
compile 'io.marioslab.basis:site:1.4'
```

You can also build the `.jar` file yourself, assuming you have Maven and JDK 1.8+ installed:
```
mvn clean install
```

The resulting `.jar` file will be located in the `target/` folder.

You can also find `SNAPSHOT` builds of the latest and greatest changes to the master branch in the SonaType snapshots repository. The snapshot is build by [Jenkins](https://libgdx.badlogicgames.com/jenkins/job/basis-site/)

To add that snapshot repository to your Maven `pom.xml` use the following snippet:

```
<repositories>
    <repository>
        <id>oss-sonatype</id>
        <name>oss-sonatype</name>
        <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
```

Google will tell you how to do the same for your Gradle builds.

## Architecture
Basis-site consists only of a handful of classes you can directly integrate in your JVM app.

If you want to extend the base functionality, it is important to understand the general architecture of basis-site.

As illustrated in the command line usage section, basis-site takes an input directory, processes the files it encounters, and writes the results to an output directory.

This entire process is encapsulated by the [`SiteGenerator`](src/main/java/io/marioslab/basis/site/SiteGenerator.java) class. The class recursively scans the input directory, and passes each input file through a configurable list of [`SiteFileProcessor`](src/main/java/io/marioslab/basis/site/SiteFileProcessor.java) instances. Input files (and folders and their children) starting with an underscore (`_`) in their file name will be skipped and not passed to the processors.

For each input file the site generator encounters, it constructs a [`SiteFile`](src/main/java/io/marioslab/basis/site/SiteFile.java) instance. A site file consists of an input file, and output file, its content (stored as a `byte[]` in the site file instance), and an optional map of metadata.

A `SiteFile` created from an input file is passed to all `SiteFileProcessor` instances in the order the processors were passed to the generator. Each processor can inspect the properties of the file, and modify the file's content and output file name. When a processor modifies a `SiteFile`, the modified site file is passed to the next processor.

When all processors have processed a file, the generator writes the final content to the output file in the output directory.

This simple architecture allows for some interesting scenarios. Say we want to minify all `.css` and `.js` files before writing them to the output folder. We can write a simple `SiteFileProcessor` that will only process `.css` and `.js` files, which it can decide based on the input file name stored in the `SiteFile`. The processor would replace the content of the site file with its minified version, and pass the site file on to the next processor in the chain.

By default, basis-site comes with a single processor called [`TemplateFileProcessor`](src/main/java/io/marioslab/basis/site/processors/TemplateFileProcessor.java). It will process any file with the infix `.bt.` in its file name and evaluate it as a [basis-template](https://gitub.com/badlogic/basis-template). The template file processor takes a list of `FunctionProvider` instances which inject functions and variables for use by the code in the template. The default implementation (as discussed in the command line usage section) is provided by `BuiltInFunctionProvider`. The input content will be replaced with the evaluation result of the template engine.

Assume we have written the fabled minifying processor. We can then combine the effects of the template and minifier processor for a common use case: merging multiple `.js` files into a single file and minifying the result.

A simple folder layout for this use case could look like this:

```
input/
    js/
        _somecode.js
        _othercode.js
        code.bt.js
    ...
```

The files `_somecode.js` and `_othercode.js` implement different functionality of your site. The start with an `_`, so they will not be copied to the output directory. The `code.bt.js` file is a templated file that pulls in the other two files:

```
{{
    include raw "_somecode.js"
    include raw "_othercode.js"
}}
```

When we run this input through the site generator, the template file processor would first evaluate the `code.bt.js` file. The end result of this step is a combined file consisting of the contents of `_somecode.js` and `_othercode.js`, and the stripping of the `.bt.` infix from the output file name. Next, the minifying processor would take the content and minify it. Finally, the generator would write the merged, minified JavaScript code to `output/js/code.js`.

All of the out-of-the-box functionality of basis-site is pulled into the `BasisSite` class, which is the driver of the command line application. In addition

## Using `BasisSite`
The [`BasisSite`](src/main/java/io/marioslab/basis/site/BasisSite.java) class is a driver that pulls together the other classes of basis-site (`SiteGenerator`, `SiteFileProcessor`] into a simple command line app, and uses [`FileWatcher`](src/main/java/io/marioslab/basis/site/FileWatcher.java) to implement watching the input directory for changes and automatically re-generating the site. If you don't want to extend the out-of-the-box functionality, this is the class to use in your JVM app.

There are two ways of embedding the class: providing it with command line arguments from which it will instantiate a `BasisSite` instance, or providing it with a `SiteGenerator` and other arguments programmatically. Here, we'll focus on construction from command line arguments.

Basis-site uses [basis-arguments](https://github.com/badlogic/basis-arguments) for command line parsing. If your app that embeds basis-site also requires command line argument parsing, it's strongly recommended to built on top of basis-arguments.

Here's the simplest example that supports file watcher mode and parses arguments for your own app in addition to the arguments `BasisSite` consumes:

```java
public static void main(String[] arguments) {
    // Creates the arguments consumed by BasisSite
    Arguments args = BasisSite.createDefaultArguments();

    // Add your own arguments
    StringArgument passwordArg = args.addArgument(new StringArgument("-p", "The password", false));

    // Parse the command line arguments and construct the BasisSite instance
    ParsedArguments parsedArgs = null;
    BasisSite site = null;
    try {
        parsedArgs = args.parse(arguments)
        site = new BasisSite(parsedArgs);
    } catch (Throwable t) {
        // Parsing the arguments or constructing the BasisSite instance failed
        Log.error(t.getMessage());
        args.printHelp();
        System.exit(-1);
    }

    // Start the generator in a separate thread, otherwise the call
    // to generate() will block this thread.
    new Thread((Runnable) () -> {
        try {
            finalSite.generate();
        } catch (Throwable t) {
            Log.error(t.getMessage());
            Log.debug("Exception", t);
        }
    }).start();

    // The rest of your app's code goes here
    String password = parsedArgs.getValue(passwordArg);
    ...
}
```

## Using `SiteGenerator` and `FileWatcher`
If you need more customization, for example adding your own `SiteFileProcessor`, it's easiest to work with `SiteGenerator` and (optionally) `FileWatcher` directly.

```java
// Create the SiteGenerator
SiteGenerator generator = new SiteGenerator(new File("input/"), new File("output/"));

// Create the template file processor, using the built-in function provider
BuiltinFunctionProvider builtinProvider = new BuiltinFunctionProvider(generator);
TempalateFileProcessor templateProcessor = new TemplateFileProcessor(Arrays.asList(builtinProvider /* Add your function providers here */));

// Add the processor to the generator. Add your own processors here.
generator.addProcessor(templateProcessor);

// Generate the initial output. You may want to delete the output folder before that (omitted).
generator.generate( (file) -> {
    Log.info("Processed " + file.getInput().getPath() + " -> " + file.getOutput().getPath());
});

// Start the file watcher. This call will block indefinitely.
FileWatcher.watch(generator.getInputDirectory(), new Runnable() {
    // if anything changed in the input directory, re-generate the output.
    // You may want to delete the output folder before that (omitted).
    generator.generate( (file) -> {
        Log.info("Processed " + file.getInput().getPath() + " -> " + file.getOutput().getPath());
    });
})
```

## Writing a `SiteFileProcessor`
Site file processors must implement the [`SiteFileProcessor](src/main/java/io/marioslab/basis/site/SiteFileProcessor.java) interface.

The `SiteFileProcessor#process(SiteFile)` method can modify the content of the input file, which is then passed on to the next processor in the chain. You can inspect the `SiteFile` to decide if you want to modify the file, e.g. based on the input file name extension. You can access the content of the file via `SiteFile#getContent()`. If the file is a text file, the `byte[]` will encode the string as UTF-8. Otherwise the content is the binary content of the file. You can set new content via `SiteFile#setContent()`.

The second method a site file processor must implement is the `SiteFileProcessor#processOutputFileName(String fileName)`. The method can return a modified version of the file name, e.g. remove an infix, or add characters. If no modification takes place, the passed in file name must be returned.

A simple `SiteFileProcessor` that remove all blank lines in `.txt` files could look like this:

```java
public class BlankLineFileProcessor implements SiteFileProcessor {
    @Override
    public void process (SiteFile file) {
        try {
            // Remove empty lines from the content
            String[] lines = new String(file.getContent(), "UTF-8").split("\\r?\\n");
            StringBuilder builder = new StringBuilder();
            for (String line : lines) {
                if (line.trim().isEmpty()) {
                    builder.append(line);
                    builder.append('\n');
                }
            }

            // Set the new content
            file.setContent(builder.toString().getBytes("UTF-8"));
        } catch (UnsupportedEncodingException e) {
            throw new SiteGeneratorException("Couldn't convert content of .txt files to UTF-8 string.", e);
        }
    }

    @Override
    public String processOutputFileName (String fileName) {
        // Simply return
        return fileName;
    }
}
```

## Writing a `FunctionProvider`
TBD

## Logging
Basis-site uses [minlog](https://github.com/esotericsoftware/minlog) for logging. Please see its documentation if you need to modify logging.

## Examples
Check out the source code of [marioslab.io](https://marioslab.io) on [GitHub](https://github.com/badlogic/marioslab-site). It combines basis-site (for static content generation) and [Javalin](https://javalin.io/) for dynamic parts (via HTTP RPC endpoints and some JavaScript).

# License
See [LICENSE](./LICENSE).

# Contributing
Simply send a PR and grant written, irrevocable permission in your PR description to publish your code under this repository's [LICENSE](./LICENSE).