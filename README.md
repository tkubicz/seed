# Rebar

**A Rust framework for creating webapps**

## Quickstart

### Setup
This package requires you to install [Rust](https://www.rust-lang.org/en-US/).

 You'll need a recent version of Rust's nightly toolchain:
`rustup update`
`rustup default nightly`,

The wasm32-unknown-unknown target:
`rustup target add wasm32-unknown-unknown --toolchain nightly`,


And wasm-bindgen: 
`cargo +nightly install wasm-bindgen-cli`


You also need an Html file that loads your app's compiled module, and provides a div with id 
to load the framework into. It also needs the following code to load your WASM module -
 Ie, the body should contain this:

```html
 <div id="main"></div>

<script>
    delete WebAssembly.instantiateStreaming;
</script>

<script src='./pkg/rebar.js'></script>

<script>
    const { render } = wasm_bindgen;

    function run() {
        render();
    }

    wasm_bindgen('./pkg/rebar_bg.wasm')
        .then(run)
        .catch(console.error);
</script>
```
Where `rebar` in `rebar.js` and `rebar_bg.wasm` is replaced by your app's name.

You will eventually need to modify this file to 
change the page's title, add a description, favicon, stylesheet etc.

Your `Cargo.toml` file needs `wasm-bindgen`, and `rebar` as depdendencies, and crate type
of `"cdylib"`. Example:

```toml
[package]
name = "appname"
version = "0.1.0"
authors = ["Name <email@email.com>"]

[lib]
crate-type = ["cdylib"]

[dependencies]
wasm-bindgen = "^0.2.28"
rebar = "^0.1.0"
```

### Minimal example

lib.rs:
```rust
extern crate rebar;
extern crate wasm_bindgen;

use wasm_bindgen::prelude::*;
use rebar::prelude::*;

// use rebar::dom_types::{Attrs, Style, El, Events, Event, Tag};
use rebar::vdom;


// The ELM Architecture (TEA)


// MODEL
// todo remove pub ?
#[derive(Clone, Debug)]
pub struct Model {
    pub clicks: i8,
    pub descrip: String,
}

impl Default for Model {
    // Initialize here, as in TEA.
    fn default() -> Self {
        Self {
            clicks: 0,
            descrip: "(Placeholder)".into(),
        }
    }
}


// UPDATE

pub enum Msg {
    Increment,
    Decrement,
    ChangeDescrip(String),
}

fn update(msg: Msg, model: Model) -> Model {
//    let model2 = model.clone(); // todo deal with this.
    match msg {
        Msg::Increment => {
            Model {clicks: model.clicks + 1, ..model}
        },
        Msg::Decrement => {
            Model {clicks: model.clicks - 1, ..model}
        },
        Msg::ChangeDescrip(descrip) => {
            Model {descrip, ..model}
        }
    }
}



// VIEW

fn comp(model: &Model) -> El<Msg> {
    let mut button = El::empty(Tag::Button);
    button.text = Some("Click me!".into());
    button.events = Events::new(vec![(Event::Click, Msg::Increment)]);

    div![ attrs!{"class" => "good elements"}, vec![
        div![ attrs!{"class" => "ok elements"},
              style!{"color" => "purple"; "border" => "2px solid #004422"},
              vec![
                  h2![ Attrs::empty(), vec![], "A walk in the woods" ],
                  h3![ Attrs::empty(), vec![], (model.clicks+1).to_string() ],
                  button,
              ], "" ],

        p![ Attrs::empty(), vec![], model.descrip.clone() ]
    ], "" ]
}


#[wasm_bindgen]
pub fn render() -> Result<(), JsValue> {
    let model = Model::default();
    vdom::mount(model, &update, &comp, "main")
}f
```

## Building and running

For details, reference [the wasm-pack documentation](https://rustwasm.github.io/wasm-pack/book/prerequisites/index.html).

To build your app, run the following two commands:

```
cargo build --target wasm32-unknown-unknown`
```
and 
```
wasm-bindgen target/wasm32-unknown-unknown/debug/rebar.wasm --no modules --out-dir ./pkg
```
where `rebar` is replaced with your app's name. This compiles your code in the target
folder, and populates the pkg folder with your WASM module, a Typescript definitions file,
and a Javascript file used to link your module from HTML.

You may wish to create a build script with these two lines. (`build.sh` for Linux; `build.ps1` for Windows)

For development, you can view your app using a dev server, or by opening the HTML file in a browser.

For example, after installing the  [http crate](https://crates.io/crates/https), run `http`.
Or with Python 3 installed, run `python -m http.server`

## Example with more extensive coverage


## Guide

### Prerequisites
**Rust**: Proficiency in Rust isn't required to get started using this framework.
It helps, but I think you'll be able to build a usable webapp using this guide,
and example code alone. For business logic behind the GUI, more study may be required.
The official [Rust Book](https://doc.rust-lang.org/book/2018-edition/index.html) is a good
place to start.

**Web fundamentals**: Experience building websites using HTML/CSS or other frameworks
is required. Neither this guide nor the API docs describes how web pages are structured,
or what different HTML/DOM elements, attributes, styles etc do. You'll need to know these before
getting started. Rebar provides tools used to assemble and manipulate these fundamentals.
Mozilla's [MDN web docs](https://developer.mozilla.org/en-US/docs/Learn)
is a good place to start.

**Other frontend frameworks** The design principles Rebar uses are similar to those
used by React, Elm, and Yew. People familiar with how to set up interactive web pages
using these tools will likely have an easy time learning Rebar.


### App structure

**Model**

**Messages**

**Update function**

**View**


### Elements, attributes, styles, and events.
When passing your layout to Rebar, attributes for DOM elements (eg id, class, src etc), 
styles (eg display, color, font-size), and
events (eg onclick, contextmenu, dblclick) are passed to DOM-macros (like div!{}) using
unique types. T

Views are described using El structs, defined in the dom_types module. They're most-easily created
with a shorthand using macros. These macros can take any combination of the following 5 argument types:
(0 or 1 of each) `Attrs`, `Style`, `Events`, `Vec<El>` (children), and `&str` (text). Attrs, Style, and Events
are most-easily created usign the following macros respectively: `attrs!{}`, `style!{}`, and `events!{}`. All
elements present must be aranged in the order above: eg `Events` can never be before `Attrs`.

For example, the following code returns an `El` representing a few dom elements displayed
in a flexbox layout:
```rust
    div![ style!{"display" => "flex"; flex-direction: "column"}, vec![
        h3![ "Some things" ],
        button![ events!{"click" => Msg::SayHi}, "Click me!" ]
    ] ]
```

The only magic parts of this are the macros used to simplify syntax for creating these
things: text are normal rust borrowed strings; children are Vecs of sub-elements; 
Attrs, Style and Events are thinly-wrapped HashMaps. They can be created independently, and
passed to the macros separately. The following code is equivalent; it uses constructors
from the El struct. Note that `El`, `Attrs`, `Style`, `Tag`, and `Event` are imported with the Rebar
prelude.


```rust
    // heading and button here show two types of element constructors
    let mut heading = El::new(
        Tag::H2, 
        Attrs::empty(), 
        Style::empty(), 
        events::Empty,
        "Some things",
        vec::New()
    );  
    
    let mut button = El::empty(Tag::Button);
    button.add_event(Event::Click, Msg::SayHi);

    let children = vec![heading, button];
    
    let mut elements = El::empty(Tag::Div);
    el.add_style("display", "flex");
    el.add_style("flex-direction", "column");
    el.children = children;
    
    el
    
```

The following equivalent example shows creating the required structs without constructors,
to demonstrate that the macros and constructors above represent normal Rust structs,
and provide insight into what abstractions they perform:
```rust
// Rust has no HashMap literal syntax; you can see why we prefer macros!
let mut style = HashMap::new();
style.insert("display", "flex");  
style.insert("flex-direction", "column");  

El {
    tag: Tag::Div,
    attrs: Attrs { vals: HashMap::new() },
    style,
    events: Events { vals: Vec::new() },
    text: None,
    children: vec![
        El {
            tag: Tag::H2,
            attrs: Attrs { vals: HashMap::new() },
            style: Style { vals: HashMap::new() },
            events: Events { vals: Vec::new() },
            text: Some(String::from("Some Things")),
            children: Vec::new()
        },
        El {
            tag: Tag::button,
            attrs: Attrs { vals: HashMap::new() },
            style: Style { vals: HashMap::new() },
            events: Events { vals: vec![(Event::Click, Msg::SayHi)] },
            text: None,
            children: Vec::new()
        } 
    ]
}

```

For most uses, the first example (using macros) will be the easiest to read and write.
You can mix in constructors (or struct literals) in components as needed, depending on your code structure.


### Components
The analog of components in frameworks like React are normal Rust functions that that return Els.
The parameters these functions take are not treated in a way equivalent
to attributes on native DOM elements, like in React; they just provide a way to 
organize your code. In practice, they feel similar to components in React, but are just
functions usedy to create elements that end up in the `children` property of
parent elements.

For example, you could break up the above example like this:
```rust
    fn text_display(text: &str) -> El<Msg> {
        h3![ text ]
    }  
    

    div![ style!{"display" => "flex"; flex-direction: "column"}, vec![
        text_display("Some things"),
        button![ events!{"click" => Msg::SayHi}, "Click me!" ]
    ] ]
```

Note that this 'component' returns a single El that is inserted into its parents'
`children` Vec; you can use this in patterns as you would in React. You can also use
functions that return Vecs or Tuples of Els, which you can incorporate into other components
using normal rust code (Eg using the extend method of the `children` Vec.) Rust's type system
ensures that only `El`s  can end up as children of other `El`s, so if your app compiles,
you haven't violated any rules.
 
Note that unlike in JSX, there's a clear syntax delineation here between natural HTML
elements, and custom components.

### Comments in the view
The Element-creation macros used to create views are normal Rust code, you can
use comments in them normally: either on their own line, or in line.

## Goals:
- Documentation that matches the current version. If you're unable to get example code working
using the latest version of framework on Crates.io, submit an issue on Github.

- Learning the syntax, creating a project, and building it should be easy - regardless
of your familiarity with Rust.

- A clean API that's easy to read, write, and understand.


### A note on the view syntax
This project takes a different approach to describing how to display DOM elements 
than others. It neither uses completely natural (ie macro-free) Rust code, nor
an HTML-like abstraction (eg JSX or templates). My intent is to make the code close 
to natural Rust, while streamlining the syntax in a way suited for creating 
a visual layout with minimal repetition. The macros used here are thin wrappers
for constructors, and don't conceal much.

The relative lack of resemblance to HTML be offputting at first, but the learning
curve is shallow, and I think the macro syntax used to create elements, attributes etc
is close-enough to normal Rust syntax that it's easy to reason about how the code
should come together, without compartmentalizing it into logic code and display code.
 This lack of separation
in particlar is a subjective, controversial decision, but I think the benefits 
are worth it.


### Where to start if you're familiar with existing frontend frameworks
The Rosetta Stone example (examples/rosetta_stone) provides equivalent code
in React, Elm, Yew, and Rebar. Comparing the code of a framework you know with
Rebar should make this easy.

### Suggestions? Critique? Submit an issue or pull request on Github


### Influences
- This project is strongly influenced by Elm, React, and Redux. The overall layout
of Rebar apps mimicks that of The Elm Architecture.

## FAQ
**Q:** There are several existing frameworks for Rust in WASM; why add another?

**A:** My goal is for this to be easy to pick up from looking at a tutorial or documentation, regardless of your
level of experience with Rust. I'm distinguising this package through clear examples
and documentation (see goals above), and using `wasm-bindgen` internally. I started this
project after being unable to get existing frameworks working,
due to lack of *standalone* examples, and inconsistency between documentation and published versions.

Rebar approaches HTML-display syntax differently from existing packages: 
rather than use an HTML-like markup similar to JSX, 
it uses Rust builtin types, thinly-wrapped by a macro for each html element.
This subjective decision may not appeal to everyone, 
but I think it integrates more naturally with the Rust language.

**Q:** Why build a frontend in Rust over Elm or Javascript-based frameworks?

**A:** You may prefer writing in Rust, and using packages from Cargo vis npm. Additionally,
wasm-based frontends are usally faster than JS-based ones. You may choose this approach over
Elm if you're already comfortable with Rust, want the performance benefits, or don't
want to code business logic in a purely-functional langauge.

**Q:** How stable is the API:

**A:** Not stable yet! Subject to change, especially DOM-element-related syntax.

## Shoutouts
 - The WASM-Bindgen team: For building the tools this project relies on
 - Alex Chrichton: For being extraodinarily helpful in the Rust / WASM community
 - The Elm team: For creating and standardizing the Elm architecture
 - Denis Kolodin: for creating the inspirational Yew framework
 