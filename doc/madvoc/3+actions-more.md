# Actions (cont.)

This page describes some more action-related features.

## Custom action method annotations

It's very common that most of your action methods will be annotated
with two-three sets of element values for `@Action` annotation. For
example, most of view actions paths may be mapped to **GET** requests,
all forms may be mapped to **\*.do** and **POST** requests, all json
actions may be mapped to **\*.json** requests. So every time when you
need to annotated a method, you will have to repeat the most of
annotation elements, over and over again.

Fortunately, *Madvoc* is clever;) It allows you to define custom
annotations with different default values. Since we can not extend
annotations, custom action annotation must contains the same elements as
the default one (`@Action`) - and even not all, but just the modified
ones.

Let's create custom annotation for all forms, that will have extension
"**.do**"" and mapped to **POST** method:

~~~~~ java
    @Retention(RetentionPolicy.RUNTIME)
    @Target({ElementType.METHOD})
    public @interface PostAction {

    	String value() default "";

    	String extension() default "do";

    	String alias() default "";

    	String method() default "POST";

    }
~~~~~

As said above, it is not necessary to repeat definition for `alias`
element. Now, when an action method is annotated with this custom
annotation:

~~~~~ java
    @PostAction
    public void someAction() {}
~~~~~

it will be equivalent to the:

~~~~~ java
    @Action(extension = "do", method = "POST")
    public void someAction() {}
~~~~~

It is obvious that custom annotation reduces the amount of repeated
code.

Finally, custom annotations are registered in `MadvocConfig`, either in
pure Java or in properties file.

## Name replacements

It is possible to reference class and method names in name values of the
*Madvoc* annotations. For example, when specifying the full action path
it is possible to reference the default extension, allowing it to be no
more hard-coded in the action string:

~~~~~ java
    @MadvocAction
    public class HelloAction {

    	@Action("/bonjour-monde.[ext]")
    	public void world() {
    	}
    }
~~~~~

In this action path value, extension \'`[ext]`\' will be replaced with
default *Madvoc* extension. The following replacements are available:

* `[package]` - replaces default package name in package-level
  annotations;
* `[class] `- replaces default class name in class-level annotations;
* `[method]` - replaces default method name in method-level annotations;
* `[ext]` - replaces default extension in \'extension\' and \'value\'
  elements of method-level annotations.

## Supplement actions

By default, when some non-registered action path is requested, *Madvoc*
returns error 404. All pages, including the static ones, must have
registered its own action method. This is especially repetitive for
static pages, such as for documentation and so on: each page would
require an action class with one default action method that returns
`void`.

*Madvoc* provides so called \'supplement actions\' for this situations.
Supplement action is an action class with default action method that
will be registered as action handler for each requested non-registered
action path that ends with default extension. First time when
non-registered action path is requested, *Madvoc* will register
supplement action for that action path.

This feature is by default turned off. It can be turned on in the
`Madvoc` configuration:

~~~~~ java
    public class MyMadvocConfig extends MadvocConfig {

    	public MyMadvocConfig() {
    		supplementAction = DefaultActionSupplement.class;
    	}
    }
~~~~~


For every non-registered action path, such as `/foo.html`, *Madvoc* will
register supplement action (here that is `DefaultActionSupplement`) as its action
handler. Since default supplement action returns `void`, in practice
that means that requesting `/foo.html` will automatically redirect to
`/foo.jsp`.

Using supplement actions is a potential critical **memory-leak** issue!
*Madvoc* will add and keep registration info for every requested
previously non-registered action path. Malicious user may request
infinite number of previously non-registered pages and for each *Madvoc*
will add new registration info. This would constantly increase used
memory by some small value, eventually ending with out of memory
exception.
{: .warn}

Because of this issue, supplement actions should be used with care.
Furthermore, similar effect can be achieved with *Madvoc* using other
techniques, such as url rewriting etc.

Note that supplement actions are only required if there is a need to
render JSP page via *Madvoc*. But there is an alternative, when just
static pages are used - simply do nothing and use HTML pages. When an
action request is not processed by *Madvoc*, then request will be simply
passed to servlet container.

## Reverse action path mapping

By default, *Madvoc* parses action method (class and/or package) to
build it's action path, during the registration. In that case it is
necessary to provide and register all action methods before starting
working with *Madvoc*. This can be done either automatically, either
manually.

*Madvoc* provides reverse action path mapping: mapping action paths to
action methods. For every requested action path, *Madvoc* may try to
find if there is a matching class and method, using simple convention
rules. If such class and method is found, they will be registered as
action method handler. Therefore, all registrations happens later,
during the work of application. This process of mapping is reverse of
default mapping.

To enable this feature, two things must be set:

~~~~~ java
    public class MyMadvocConfig extends MadvocConfig {

    	public MyMadvocConfig() {
    		actionPathMappingEnabled = true;
    		setRootPackageOf(IndexAction.class);
    	}
    }
~~~~~

The following action path mapping convention id defined by
`ActionPathMapper`:

| actionPath        | action signature       |
|-------------------|------------------------|
| /zoo/boo.foo.html | zoo.Boo#foo            |
| /zoo/boo.foo.ext  | zoo.Boo#fooExt         |
| /zoo/boo.html     | zoo.Boo#view           |
| /zoo/boo.ext      | zoo.Boo#viewExt        |
| /zoo/boo          | zoo.Boo#execute        |


Here \'html\' denote default extension, \'ext\' denote any non-default
extension. Method names \'view\' and \'execute\' are actually first and
second default action name, defined in global *Madvoc* configuration. Of
course, if there are less or none default action name, *Madvoc* will
fail-back to the existing name.
