# Jerry ![jerry](jerry-big.png){: .float-left}

@@javadoc-p(jerry)

*Jerry* is a [jQuery][1] in Java. *Jerry* is a fast and
concise Java Library that simplifies HTML document parsing, traversing
and manipulating. *Jerry* is designed to change the way that you parse
HTML content.

*Jerry* belongs to <var>jodd-lagarto</var> module and has dependency on
[SLF4J](/doc/logging-jodd.html).

## What does Jerry code look like?

~~~~~ java
    import static jodd.jerry.Jerry.jerry;
    ...
    Jerry doc = jerry(html);
    doc.$("div#jodd p.neat").css("color", "red").addClass("ohmy");
~~~~~

## How Jerry works?

We tried to keep *Jerry* API identical to jQuery as much as possible; in
some cases you can simply copy some jQuery code and paste it in Java! Of
course, there are some differences due to different environments code is
executed in.

### Creating document

In Java we do not have the document context as in browsers and
javascript, so we need to create one first. To do that, simply pass HTML
content to *Jerry* static factory method. That will create a root
*Jerry* set, containing a `Document` root node of the parsed DOM tree.

What happens in the background is that *Lagarto* parser is invoked to
build a DOM tree.

*Jerry* uses *Lagarto DOM* parser for parsing the content and building
the DOM tree.
{: .example}

### Using CSS selectors

![csselly](../csselly/csselly.png){: align="left" style="margin-right:10px;"}
You can use most of standard CSS selectors
and also most of the jQuery CSS selectors extensions. CSS selectors are
supported by [*CSSelly*](/doc/csselly).

### Differences

As *Jerry* speaks Java, there are some differences in API made to make
*Jerry* API more Java friendly. For example, `css()` method accepts an
array of property/values, and not a single string.

~~~~~ java
    jerry(html).$("tr:last").css("background-color", "yellow", "fontWeight", "bolder");
~~~~~

Similarly, `each()` method receives a callback instance and it is not so
fluent as in javascript;):

~~~~~ java
    doc.$("select option:selected").each(new JerryFunction() {
    		public boolean onNode(Jerry $this, int index) {
    			str.append($this.text()).append(' ');
    			return true;
    		}
    	});
~~~~~

### Unsupported stuff

As *Jerry* is all about **static** manipulation of HTML content, all
jQuery methods and selectors that are related to any dynamic activity
are not supported. This includes animations, ajax calls, selectors that
depends on CSS definitions...

## Configuration

*Jerry* internally uses [*Lagarto DOM*](/doc/lagarto) builder to parse
input content and to produce HTML code. The builder is configurable and
supports predefined parsing modes for HTML, XHTML and XML.

By default, *Jerry* uses the builder in HTML parsing mode. Here is an
example how to change the predefined parsing mode:

~~~~~ java
    Jerry jerry = jerry().enableHtmlMode().parse(html);
~~~~~

To configure it more, we can use the following snippet:

~~~~~ java
    JerryParser jerryParser = Jerry.jerry();
    jerryParser.getDOMBuilder().setIgnoreComments(true);
    Jerry jerry = jerryParser.parse(xhtml);
~~~~~

Check all details about [configuration and parsing modes](/doc/lagarto/lagarto-properties.html) for *Lagarto*,
parser used by *Jerry*.

## Examples

Following examples depend on content of live web pages. At some point in
time pages may change in such way that the examples stop working.
{: .attn}

### New music releases from allmusic.com

Site [allmusic.com][2] shows new music releases in the
right column. Here is a simple code that downloads the page, parse it
and displays all releases in console:

~~~~~ java
    public class AllMusicNewReleases {

    	public static void main(String[] args) throws IOException {

    		// download the page super-efficiently
    		File file = new File(SystemUtil.getTempDir(), "allmusic.html");
    		NetUtil.downloadFile("http://allmusic.com", file);

    		// create Jerry, i.e. document context
    		Jerry doc = Jerry.jerry(FileUtil.readString(file));

    		// parse
    		doc.$("div#new_releases div.list_item").each(new JerryFunction() {
    			public boolean onNode(Jerry $this, int index) {
    				System.out.println("-----");
    				System.out.println($this.$("div.album_title").text());
    				System.out.println($this.$("div.album_artist").text().trim());
    				return true;
    			}
    		});
    	}
    }
~~~~~

Nice :)

### Change Google page

Let's remove toolbar from Google page and remove Google logo image with
simple HTML text.

~~~~~ java
    public class ChangeGooglePage {

    	public static void main(String[] args) throws IOException {

    		// download the page super-efficiently
    		File file = new File(SystemUtil.getTempDir(), "google.html");
    		NetUtil.downloadFile("http://google.com", file);

    		// create Jerry, i.e. document context
    		Jerry doc = Jerry.jerry(FileUtil.readString(file));

    		// remove div for toolbar
    		doc.$("div#mngb").detach();
    		// replace logo with html content
    		doc.$("div#lga").html("<b>Google</b>");

    		// produce clean html...
    		String newHtml = doc.html();
    		// ...and save it to file system
    		FileUtil.writeString(
    			new File(SystemUtil.getTempDir(), "google2.html"),
    			newHtml);
    	}
    }
~~~~~

Easy peasy!

### Facebook bot

To demonstrate the power of *Jerry*, we created a little Facebook bot
just for fun:) The task was to create a bot that will login to Facebook
account, list friends proposals and send a few "Add friend"
requests. To see how, [read it here.](facebook-bot.html)


### Jerry parses POM XML (too)!

*Jerry* can be used to parse XML files, too! We needed to parse Maven
POM files, in order to display dependencies on our download page. First,
we had to enable the XML mode of *Lagarto* parser:

~~~~~ java
	Jerry.JerryParser jerryParser = new Jerry.JerryParser();
	jerryParser.enableXmlMode();
	Jerry doc = jerryParser.parse(FileUtil.readString(pomFile));
~~~~~

and then we can access the content via CSS selectors, for example:

~~~~~ java
	Jerry dependencies = doc.$("dependencies dependency");

	dependencies.each(new JerryFunction() {
		@Override
		public boolean onNode(Jerry $this, int index) {
			// skip test dependencies
			if ($this.$("scope").text().equals("test")) {
				return true;
			}

			String artifactId = $this.$("artifactId").text();
			String optionalStr = $this.$("optional").text();

			// process

			return true;
		}
	});
~~~~~

Complete code you can find [here][3].

### Jerry on Groovy!

*Jerry* can be used very nicely in Groovy! [Rob Flatcher][4] uses it in Groovy tests like this (snippet):

~~~~~ java
    import static jodd.jerry.Jerry.jerry as $

    def dom = $(output)

    def labels = dom.find('label')
    labels*.attr('for') == ['foo_hours', 'foo_minutes', 'foo_seconds']
    labels*.text() == ['\u00a0hours ', '\u00a0minutes ', '\u00a0seconds ']
    labels.every {
    	it.find(':input').attr('id') == it.attr('for')
    }
~~~~~

For more details, check unit tests in [grails-joda-time][5].

[1]: http://jquery.com/
[2]: http://allmusic.com
[3]: http://github.com/oblac/tools/blob/master/src/jodd/tools/modlist/PomList.java
[4]: http://freeside.co/
[5]: https://github.com/gpc/grails-joda-time
