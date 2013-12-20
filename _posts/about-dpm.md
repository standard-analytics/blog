{{{
	"title" : "Science 3.0: science for machines",
	"tags": [ "blog" ],
	"category" : "announcement",
	"date" : "12-20-2013",
    "author": "Sebastien"
}}}

What if we could remove all of the frills and ceremony from scientific
publishing and just make the data, methods and results available, easy
to re-use, and verifiably reproducible? JavaScript developers working
on a new project have [npm](http://npmjs.org) to do just this - they
can simply specify other existing programs that the project depends on
(much like scientists want to reference other published papers) and
these 'dependencies' are installed with their new project using
[npm](http://npmjs.org). Amazing, and really fun too. In science
today, this workflow simply does not exist.  This is because references
('dependencies') are in static, often proprietary formats.  Additionally,
incorporating raw data from other studies requires manual extraction. Finally, 
re-implementing previously published methods
is rarely possible, as publishing data and analytics in a
standardized, linked format is not common practice.

![software development vs science](img/dev_v_science.png)

At Standard Analytics we package science in an accessible way and make
it reusable so that scientists - just like software developers - can
build new and world-changing things on the back of all of the work
that has come before. As former academic scientists ourselves, we
couldn't be more excited to take the first steps toward making this
dream a reality and we're pleased to introduce you to ```dpm2``` the
package manager for data and science.

To find out more about ```dpm2``` and take it for a test run, check
it out on [github](https://github.com/standard-analytics/dpm2).

[![NPM](https://nodei.co/npm/dpm2.png)](https://nodei.co/npm/dpm2/)

```dpm2``` is largely inspired by [npm](http://npmjs.org). It
leverages the same technologies ([node.js](http://nodejs.org/) and
[CouchDB](http://couchdb.apache.org/)), re-uses a lot of the
[npm](https://github.com/isaacs/npm) dependencies _but_ differs from
npm in a very important way: we store
[data packages](http://dataprotocols.org/data-packages/) so that
- Each piece of data has its own [URL](http://en.wikipedia.org/wiki/Uniform_resource_locator), and
- Powerful semantic search can be performed.

Let's see in practice why this matters with a detailed walk through, using an example.

One of the latest blog posts of
[Mode Analytics](http://blog.modeanalytics.com/), presents an analysis
on whether Stanford graduates are good investments in the startup
world. This blog is truly remarkable, in that all the data that is
discussed is openly shared on
[their Github account](https://github.com/mode/blog).  Let's package
these data so that they can be easily re-used _ala npm_.

For that we create a [data package](http://dataprotocols.org/data-packages/).

A [csv](http://en.wikipedia.org/wiki/Comma-separated_values) file
```school.csv``` containing the data of interest

```
"Schools","Founders","Unicorns"
"Stanford",482,13
"Harvard",487,8
"Berkeley",158,5
...
```

will be packaged like that:

```
{
  "name": "unicorns",
  "version": "0.0.0",
  "description": "Data used in Mode Analytics Stanford founders blog post",
  "sources": [{
    "name": "Mode Analytics blog",
    "web": "https://github.com/mode/blog/tree/master/2013-12-06%20Stanford%20Founders"
  }],
  "license": {
    "type": "DbCL-1.0",
    "url": "http://opendatacommons.org/licenses/dbcl/1.0/"
  },
  "repositories": [
    {
      "type": "git",
      "url": "https://github.com/standard-analytics/blog.git",
      "path": "data/unicorns"
    }
  ],
  "keywords": ["schools", "graduates", "startup", "unicorn"],
  "author": "Sebastien Ballesteros <sebastien@standardanalytics.io>",
  "resources": [
    {
      "name": "schools",
      "path": "data/schools.csv",
      "format": "csv",
      "schema": {
        "fields": [
    	  {
    	    "name": "Schools",
    	    "description": "American top schools",
    	    "type": "number"
    	  },
    	  {
    	    "name": "Founders",
    	    "description": "Number of startup founders originating from each school",
    	    "type": "number"
    	  },
    	  {
    	    "name": "Unicorns",
    	    "description": "Number of unicorn founders originating from each school",
    	    "type": "number"
    	  }
        ]
      }
    }
  ]
}
```

This ```package.json``` file, contains just enough information so that
humans _or machines_ can happily consume the data.

Then we publish it on the Standard Analytics
[data registry](https://github.com/standard-analytics/data-registry).

<pre><code class="bash">$ dpm2 publish</code></pre>

Now, if I want to investigate if Stanford graduates founded
significantly more
[Unicorns](http://techcrunch.com/2013/11/02/welcome-to-the-unicorn-club/)
(>1B$ valuation startups) than graduates of other universities, I can
simply list this data package as ```dataDependencies``` of a new
package.json which for now contains nothing else.

```
{
  "name": "stanford-founders",
  "version": "0.0.0",
  "description": "Unicorns founders and schools origin",
  "dataDependencies": {
    "unicorns": "0.0.0"
  }
}
```

Running

<pre><code class="bash">$ dpm2 install --cache</code></pre>

will resolve all the dataDependencies and give me the data I need to
perform my analysis.

A small note on the ```--cache``` option here: without it, ```dpm2```
only retrieves a package.json where all the resources data (our csv
here) have been replaced by
[URLs](http://en.wikipedia.org/wiki/Uniform_resource_locator). This is
handy for big data when you might prefer to analyze data as a
[stream](http://nodejs.org/api/stream.html), or simply when you just
want a small subset of all the resources listed in a data package.
But with the ```--cache``` option you get all the data!

Having the data I need, I can fire up [R](http://www.r-project.org/)
and ask, _have Stanford grads founded significantly more unicorns than
Harvard ones?_


<pre><code class="r">schools <- read.csv("data_modules/unicorns/data/schools.csv")
stanford <- schools$Unicorns[schools$Schools == "Stanford"]
harvard <- schools$Unicorns[schools$Schools == "Harvard"]
prop.test(stanford, stanford + harvard, alternative = "greater")</code></pre>


But let's not stop here. Statistical results are data after all and
therefore we can package them too! Even better, the vocabulary for
statistics is _well defined_ and _bounded_ which means that we can use
semantic technologies (namely, [JSON-LD](http://json-ld.org) and
[alps.io](http://alps.io/spec/index.html)) to make it completely clear (to
both humans and machines) which
[statistical analytics](https://github.com/standard-analytics/Schemas)
were used.

While we are at it, we can also indicate as metadata how the results
were obtained so that anyone (human or machine) can _fully reproduce_
our analysis. So let's add our finding to our previous package.json
(that we originally used to get the dataDependencies).


```
{
  "name": "unicorns-schools",
  "version": "0.0.0",
  "description": "Unicorns founders and schools origin",
  "license": {
    "type": "DbCL-1.0",
    "url": "http://opendatacommons.org/licenses/dbcl/1.0/"
  },
  "repositories": [
    {
      "type": "git",
      "url": "https://github.com/standard-analytics/blog.git",
      "path": "data/unicorns-schools"
    }
  ],
  "keywords": ["stanford", "grads", "startup", "unicorns"],
  "author": "Sebastien Ballesteros <sebastien@standardanalytics.io>",
  "dataDependencies": {
    "unicorns": "0.0.0"
  },
  "analytics": [
    {
      "name": "propTest",
      "description": "Do Stanford Grads found significantly more Unicorns (>1B$ valuation startups) than other graduates?",
      "scripts": { "start": "scripts/propTests.R" },
      "inputs":  [ { "datapackage": "unicorns", "resource": "schools" } ],
      "outputs": [ { "resource": "stanfordVsHarvard" } ]
    }
  ],
  "resources": [
    {
      "name": "stanfordVsHarvard",
      "@context": {
        "@vocab": "https://raw.github/com/standard-analytics/Schemas/master/"
      },
      "@type": "propTest",
      "description": "Do Stanford grads found significantly more unicorns than Harvard ones?",
      "line": 4,
      "data": {
        "props": {
          "p": 0.61905
        },
        "chisqTest": {
          "X2": 0.7619,
          "df": 1,
          "pValue": 0.19137
        }
      }
    }
  ]
}
```

We can now publish this new ```package.json``` on the Standard
Analytics [registry](https://github.com/standard-analytics/data-registry).

<pre><code class="bash">$ dpm2 publish</code></pre>

With such a file, anyone can _verify_ the results, plus there is more:
anyone can now **quote** these finding! Let's illustrate this last
point.

As I said before, every resource of a data package published has its
own URL. So I can use the following URL
[https://registry.standardanalytics.io/unicorns-schools/0.0.0/stanfordVsHarvard](https://registry.standardanalytics.io/unicorns-schools/0.0.0/stanfordVsHarvard)
anytime I want to give my text or comments some statistical backbone.

For instance, I can precisely quote that:

> Although Stanford grads have founded more Unicorns than Harvard ones,
> [there is no evidence to show that this difference is truly significant](https://registry.standardanalytics.io/unicorns-schools/0.0.0/stanfordVsHarvard).


And the discussion could progress from there, based on concrete,
transparent, and constructive *quantitative* elements.

Last, let me stress that by simply walking down the URL, it is
possible to go from an analytic (here a
[p-value](http://en.wikipedia.org/wiki/P-value)) all the way back to
the original data.

- As we have seen, the p-value is here: [https://registry.standardanalytics.io/unicorns-schools/0.0.0/stanfordVsHarvard](https://registry.standardanalytics.io/unicorns-stanford/0.0.0/stanfordVsHarvard)

- The latest version of the full data package is here: [https://registry.standardanalytics.io/unicorns-schools/0.0.0](https://registry.standardanalytics.io/unicorns-stanford/0.0.0)

- Finally, you can see every published version of the work here: [https://registry.standardanalytics.io/unicorns-schools](https://registry.standardanalytics.io/unicorns-stanford)


To us, this is the beginning of a _new form of arguing, backed by
reproducible, quantitative evidence_.

Right now, the full API of the registry is described
[here](https://github.com/standard-analytics/data-registry). That
being said we are working to add some
[hypermedia controls](http://www.markus-lanthaler.com/hydra/) so that
reading the API doc becomes a thing of the past.


You may be thinking that generating these ```package.json``` is too
much hassle, and that scientists and data-enthusiasts have better
things to do than write such ```package.json``` files. Having been
there ourselves, we could not agree more: this has to be a **fully
effortless** process. In the next post we will introduce ```sloth```
our very own command line tool to _automatically_ generate these
files.
