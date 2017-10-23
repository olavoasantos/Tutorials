# Creating a Note-taking app with Vue.js and Unite.js
In this example, lets create a simple note-taking app, called Annotate, using TDD with Unite.js. Here, I’m going to focus on the functional part and I’m not going to worry about styling.
## Project root
To start, lets create a project folder. If you use the terminal:
```mkdir annotate && cd annotate```
And to start npm, run:
```npm init -y```
If everything went well, you should get something like this:
```
Wrote to {PATH TO PROJECT}\annotate\package.json:
{
“name”: “annotate”,
“version”: “1.0.0”,
“description”: “”,
“main”: “index.js”,
“scripts”: {
“test”: “echo \”Error: no test specified\” && exit 1"
},
“keywords”: [],
“author”: “”,
“license”: “ISC”
}
```
Opening up the folder on VS Code, you should have something like this:
// Blank project
## Installing dependencies
For this project, we’ll be using `vue.js` and `unite.js`. So, through the terminal, let’s pull in `unite.js`:
```npm install –save-dev unite.js```
This package pulls everything we need to start our development.
## Setting up the test environment
Now, open up your `./package.json` file and change the `scripts` object to:
```json
// ./package.json
“scripts”: {
“test”: “unite”
},
```
Now, we need to create our components folder. Let’s create a `src` folder where our component files will be located:
```mkdir src```
Finally, we need to tell `unite.js` to use this `src` directory as the testing folder. To do this, let’s create a `unite.config.js` file on the project root:
```touch unite.config.js```
Now, open it on your editor and add:
```js
// ./unite.config.js
module.exports = {
path: “./src”
}
```
Now, if you open up your terminal and run `npm test` you should get:
// npm test first run
## Annotate SFC tests
Now that we have our testing environment setup, let’s create our component file in our `src` folder:
```touch src/annotate.tests.vue```
Open our `annotate.tests.vue` file on you editor and add the basic component structure:
```vue
// src/annotate.tests.vue
<template>
</template>
<script>
export default {
}
</script>
<tests>
</tests>
```
To make sure everything is working correctly, let’s create two tests in our `tests` tag : a passing test and a failing test:
```vue
// src/annotate.tests.vue
<template>
<div class=”annotate”>Was mounted correctly</div>
</template>
<script>
export default {
}
</script>
<tests>
let wrapper;
/**
* It mounts the component before each test
*/
Unite.beforeEachTest(() => {
wrapper = Unite.$mount($component);
});
/** @test */
Unite.test(“It is a valid test”, () => {
expect(wrapper.text()).toEqual(“Was mounted correctly”);
});
/** @test */
Unite.test(“It is an invalid test”, () => {
expect(wrapper.text()).toEqual(“This will fail”);
});
</tests>
```
Let’s go through the code:
+ The ` template` tag contains a single div element with the text “Was mounted correctly”
+ The `tests` tag contains:
+ First we define a `wrapper` variable, which we will use to mount our component
+ The `Unite.beforeEachTest()` defines a function that will run before each test. Here, we can mount our component before each test.
+ `Unite.$mount`: This function is the `vue-test-utils` mount function, which mounts a vue component and returns a `Wrapper`
+ The `$component` is our vue component defined as variable by `unite.js` inside `*.tests.vue` files
+ The `Unite.test()` defines a new test. It accepts two parameter:
+ `name`: The test name
+ `callback`: The test function
+ `@test: It is a valid test`
+ Using `wrapper.text()` we have access to the components text content (i.e. “Was mounted correctly”)
+ Using the `expect` library, we expect that `wrapper.text()` is equal to “Was mounted correctly”
+ `callback`: The test function
+ `@test: It is an invalid test`
+ Using `wrapper.text()` we have access to the components text content (i.e. “Was mounted correctly”)
+ Using the `expect` library, we expect that `wrapper.text()` is equal to “This will fail”
Now if we run `npm test` and you should get something like this:
// Annotate.tests.vue sanity check
As expected, we get one failing test. Now that we know everything is working, let’s start the fun part and create our component!!
## Creating Annotate using TDD
### Deciding the component structure
The first thing we need to do is decide how we want our component to work. Ultimately, the basic functionality we want should include:
+ Notes with a title and a body
+ A list of notes
+ A form to create/edit notes
+ When a new note is created, it should be added to the list
+ When a note is clicked, we should see it in the form
+ When a note is edited and saved, it should update the respective note in the list
+ When a `new` button is clicked, we should see a blank form
Let’s get this to work in the simplest way we can and later, if we want, we can refactor.
### Note object
As we described, each note should contain a `title` and a `body`. For simplicity, the `note` object will be:
```js
{
id: 1,
title: “Note title”,
body: “Note body”
}
```
### List of notes
Let’s think about the list of notes:
+ It should be an array containing `note` objects
+ Each `note` object should be displayed inside a `.notes` element in the template
+ Each `.notes` child should display its `title`
With that in mind, let’s create a test asserting that, for each `note` object, we should see a child of `.notes` containing the `note`’s title:
```vue
// src/annotate.tests.vue
<tests>
// Other tests…
/** @test */
Unite.test(“It displays all the notes”, () => {
addNote(“Note 1”, “Nody body”);
addNote(“Note 2”, “Nody body”);
let notes = wrapper.find(“.notes”);
expect(notes.vnode.children.length).toEqual(2);
});
/** @helper It adds a new Note */
let addNote = (title, body) => {
let noteKey = wrapper.vm.notes.length;
wrapper.vm.notes.push({
id: (new Date).getTime(),
title: title,
body: body
});
wrapper.update();
return wrapper.vm.notes[noteKey];
};
</tests>
```
Let’s go through the code:
+ `@test: It displays all the notes`:
+ `addNote(“Note 1”, “Nody body”);`: It adds a new note using the `addNote` helper (see below)
+ `addNote(“Note 2”, “Nody body”);`: It adds a new note using the `addNote` helper (see below)
+ `let notes = wrapper.find(“.notes”);`: It finds the `.notes` DOM element
+ `expect(notes.vnode.children.length).toEqual(2);`: It expects that the `.notes` DOM element contains two children
+ `@helper: addNote(title, body)`:
+ `let noteKey = wrapper.vm.notes.length;`: It gets the current length of the `notes` array
+ `wrapper.vm.notes.push()`: Pushes a new note to the `notes` array:
+ `id: (new Date).getTime(),`: It assigns a unique `id` to the note (to be used as the `key` on the v-for loop)
+ `title: title,`: It sets the `note` object `title` to the given `title`
+ `body: body`: It sets the `note` object `body` to the given `body`
+ `wrapper.udpate();`: Triggers vue to update the DOM
+ `return wrapper.vm.notes[noteKey];`: It returns the latest `note` object
Time to get our hands dirty!!
1. Running `npm test`:
```
1) annotate.tests.vue @ It displays all the notes
TypeError: Cannot read property ‘push’ of undefined
```
This means that we’re trying to use `push` on an undefined variable. On our code, `push` is being used on the `addNote` helper to push a `note` object to the `notes` property:
```js
wrapper.vm.notes.push({
id: (new Date).getTime(),
title: title,
body: body
});
```
So let’s create the `notes` property:
```vue
// src/annotate.tests.vue
<script>
export default {
data() {
return {
notes: []
};
}
}
</script>
```
2. Running `npm test`:
```
1) annotate.tests.vue @ It displays all the notes
TypeError: Cannot read property ‘children’ of undefined
```
Now we’re trying to read the property `children` on an undefined variable. This means that `vue-test-utils` couldn’t find the `.notes` element.
Let’s create the `.notes`:
```vue
// src/annotate.tests.vue
<template>
<div class=”annotate”>
<ul class=”notes”></ul>
</div>
</template>
```
3. Running `npm test`:
```
1) annotate.tests.vue @ It displays all the notes
TypeError: Cannot read property ‘length’ of undefined
```
Now, we’re trying to get the length of `children`, but there are none. In this case, `vue-test-utils` sets `children` as undefined. So let’s create a v-for loop for the notes:
```vue
// src/annotate.tests.vue
<template>
<div class=”annotate”>
<ul class=”notes”>
<li class=”note” v-for=”note in notes” :key=”note.id” v-text=”note.title”></li>
</ul>
</div>
</template>
```
4. Running `npm test`:
We get green! Our component should look something like:
```vue
// src/annotate.tests.vue
<template>
<div class=”annotate”>
<ul class=”notes”>
<li class=”note” v-for=”note in notes” :key=”note.id” v-text=”note.title”></li>
</ul>
</div>
</template>
<script>
export default {
data() {
return {
notes: []
};
}
}
</script>
<tests>
let wrapper;
/**
* It mounts the component before each test
*/
Unite.beforeEachTest(() => {
wrapper = Unite.$mount($component);
});
/** @test */
Unite.test(“It displays all the notes”, () => {
addNote(“Note 1”, “Nody body”);
addNote(“Note 2”, “Nody body”);
let notes = wrapper.find(“.notes”);
expect(notes.vnode.children.length).toEqual(2);
});
/** @helper It adds a new Note */
let addNote = (title, body) => {
let noteKey = wrapper.vm.notes.length;
wrapper.vm.notes.push({
id: (new Date).getTime(),
title: title,
body: body
});
wrapper.update();
return wrapper.vm.notes[noteKey];
};
</tests>
```
### A form to create/edit notes
Now, we need a form to create and edit our notes. So let’s create a test asserting that it sees a form containing inputs for the title and the body:
```vue
// src/annotate.tests.vue
<tests>
// Other tests…
/** @test */
Unite.test(“It sees the form to create/edit notes”, () => {
let form = wrapper.find(“.form”);
expect(form.find(“input[name=id]”).exists()).toBe(true);
expect(form.find(“input[name=title]”).exists()).toBe(true);
expect(form.find(“textarea[name=body]”).exists()).toBe(true);
expect(form.find(“button”).exists()).toBe(true);
});
</tests>
```
Let’s go through the code:
+ `@test: It sees the form to create/edit notes`:
+ `wrapper.find(“.notes”)`: It gets the `.form` wrapper
+ `expect(form.find(“input[name=id]”).exists()).toBe(true)`: We expect that an input with a name of `id` exists
+ `expect(form.find(“input[name=title]”).exists()).toBe(true)`: We expect that an input with a name of `title` exists
+ `expect(form.find(“textarea[name=body]”).exists()).toBe(true)`: We expect that a textarea with a name of `body` exists
+ `expect(form.find(“button”).exists()).toBe(true)`: We expect that a button exists
Let’s code!
1. Running `npm test`:
```
1) annotate.tests.vue @ It sees the form to create/edit notes
Error: [vue-test-utils]: find did not return .form, cannot call find() on empty Wrapper
```
`vue-test-utils` couldn’t find a `.form` element. Let’s add that:
```vue
// src/annotate.tests.vue
<template>
<div class=”annotate”>
<ul class=”notes”>
<li class=”note” v-for=”note in notes” :key=”note.id” v-text=”note.title”></li>
</ul>
<div class=”form”>
</div>
</div>
</template>
```
2. Running `npm test`:
```
1) annotate.tests.vue @ It sees the form to create/edit notes
expect(received).toBe(expected)
Expected value to be (using ===):
true
Received:
false
```
Here, our expectations are failing! So let’s go ahead and create our inputs:
```vue
// src/annotate.tests.vue
<template>
<div class=”annotate”>
<ul class=”notes”>
<li class=”note” v-for=”note in notes” :key=”note.id” v-text=”note.title”></li>
</ul>
<div class=”form”>
<input type=”hidden” name=”id”>
<input type=”text” name=”title”>
<textarea name=”body”></textarea>
<button>Save</button>
</div>
</div>
</template>
```
3. Running `npm test`:
And we get green! At this point our component file should look something like:
```vue
// src/annotate.tests.vue
<template>
<div class=”annotate”>
<ul class=”notes”>
<li class=”note” v-for=”note in notes” :key=”note.id” v-text=”note.title”></li>
</ul>
<div class=”form”>
<input type=”hidden” name=”id”>
<input type=”text” name=”title”>
<textarea name=”body”></textarea>
<button>Save</button>
</div>
</div>
</template>
<script>
export default {
data() {
return {
notes: []
};
}
}
</script>
<tests>
let wrapper;
/**
* It mounts the component before each test
*/
Unite.beforeEachTest(() => {
wrapper = Unite.$mount($component);
});
/** @test */
Unite.test(“It displays all the notes”, () => {
addNote(“Note 1”, “Nody body”);
addNote(“Note 2”, “Nody body”);
let notes = wrapper.find(“.notes”);
expect(notes.vnode.children.length).toEqual(2);
});
/** @test */
Unite.test(“It sees the form to create/edit notes”, () => {
let form = wrapper.find(“.form”);
expect(form.find(“input[name=id]”).exists()).toBe(true);
expect(form.find(“input[name=title]”).exists()).toBe(true);
expect(form.find(“textarea[name=body]”).exists()).toBe(true);
expect(form.find(“button”).exists()).toBe(true);
});
/** @helper It adds a new Note */
let addNote = (title, body) => {
let noteKey = wrapper.vm.notes.length;
wrapper.vm.notes.push({
id: (new Date).getTime(),
title: title,
body: body
});
wrapper.update();
return wrapper.vm.notes[noteKey];
};
</tests>
```
### When a new note is created, it should be added to the list
Now that we have our form, if we fill out the form and click on the `button`, we expect that our note is added to the `notes` list. Let’s write the test:
```vue
// src/annotate.tests.vue
<tests>
// Other tests…
/** @test */
Unite.test(“It adds a new note when the button is clicked”, () => {
type(“input[name=title]”, “New Note”);
type(“input[name=title]”, “New Note”);
expect(wrapper.find(“.notes”).isEmpty()).toBe(true);
wrapper.find(“button”).trigger(“click”);
expect(wrapper.find(“.notes”).vnode.children.length).toEqual(1);
expect(wrapper.find(“.note:first-child”).text()).toEqual(“New Note”);
});
/** @helper It types into an input */
let type = (selector, value) => {
let node = wrapper.find(selector);
node.element.value = value;
node.trigger(“input”);
};
</tests>
```
+ `@test: It adds a new note when the button is clicked`:
+ `type(“input[name=title]”, “New Note”);`: Type into the `title` input “New Note”
+ `type(“input[name=title]”, “New Note”);`: Type into the `body` input “Note’s body”
+ `expect(wrapper.find(“.notes”).isEmpty()).toBe(true);`: Expect that the `.notes` DOM element does not contain any children
+ `wrapper.find(“button”).trigger(“click”)`: Clicks on the button
+ `expect(wrapper.find(“.notes”).vnode.children.length).toEqual(1);`: Expect that `.notes` DOM element has one child
+ `expect(wrapper.find(“.note:first-child”).text()).toEqual(“New Note”);`: Expect that first `.note` child DOM element has a text content of “New Note”
+ `@helper: type(selector, value)`:
+ `let node = wrapper.find(selector);`: It finds the element with the given selector
+ `node.element.value = value;`: It sets the element’s value with the given value
+ `node.trigger(“input”);`: It triggers vue’s `input` event
Let’s go through the errors to implement this!
1. Run `npm test`:
```
1) annotate.tests.vue @ It adds a new note when the button is clicked
expect(received).toEqual(expected)
Expected value to equal:
1
Received:
0
```
We expected that the note would be created when the button was clicked and a child element of `.notes` would be created. But right now, nothing happend.
First thing we need to do is trigger a function when the `button` is clicked:
```vue
// src/annotate.tests.vue
<template>
<div class=”annotate”>
<ul class=”notes”>
<li class=”note” v-for=”note in notes” :key=”note.id” v-text=”note.title”></li>
</ul>
<div class=”form”>
<input type=”hidden” name=”id”>
<input type=”text” name=”title”>
<textarea name=”body”></textarea>
<button @click=”save”>Save</button>
</div>
</div>
</template>
```
Let’s create the `save` method:
```vue
// src/annotate.tests.vue
<script>
export default {
data() {
return {
notes: []
};
},
methods: {
save() {
}
}
}
</script>
```
To save the note, first we need to know the input content. Let’s create a `note` variable in the vue instance and add v-models to the inputs:
```vue
// src/annotate.tests.vue
<template>
<div class=”annotate”>
<ul class=”notes”>
<li class=”note” v-for=”note in notes” :key=”note.id” v-text=”note.title”></li>
</ul>
<div class=”form”>
<input type=”hidden” name=”id” v-model=”note.id”>
<input type=”text” name=”title” v-model=”note.title”>
<textarea name=”body” v-model=”note.body”></textarea>
<button @click=”save”>Save</button>
</div>
</div>
</template>
<script>
export default {
data() {
return {
note: {
id: “”,
title: “”,
body: “”
},
notes: []
};
},
methods: {
save() {
}
}
}
</script>
```
Now, the `save` method can can push the new `note` to the notes list:
```vue
// src/annotate.tests.vue
<script>
export default {
data() {
return {
note: {
id: “”,
title: “”,
body: “”
},
notes: []
};
},
methods: {
save() {
this.note.id = (new Date).getTime();
this.notes.push(this.note);
}
}
}
</script>
```
+ `@methods: save()`:
+ `this.note.id = (new Date).getTime();`: It sets a unique id to the new `note` object
+ `this.notes.push(this.note);`: It pushes the new `note` object to the notes list
2. Run `npm test`:
We should now get green and our component should look something like this:
```vue
// src/annotate.tests.vue
<template>
<div class=”annotate”>
<ul class=”notes”>
<li class=”note” v-for=”note in notes” :key=”note.id” v-text=”note.title”></li>
</ul>
<div class=”form”>
<input type=”hidden” name=”id” v-model=”note.id”>
<input type=”text” name=”title” v-model=”note.title”>
<textarea name=”body” v-model=”note.body”></textarea>
<button @click=”save”>Save</button>
</div>
</div>
</template>
<script>
export default {
data() {
return {
note: {
id: “”,
title: “”,
body: “”
},
notes: []
};
},
methods: {
save() {
this.note.id = (new Date).getTime();
this.notes.push(this.note);
}
}
}
</script>
<tests>
let wrapper;
/**
* It mounts the component before each test
*/
Unite.beforeEachTest(() => {
wrapper = Unite.$mount($component);
});
/** @test */
Unite.test(“It displays all the notes”, () => {
addNote(“Note 1”, “Nody body”);
addNote(“Note 2”, “Nody body”);
let notes = wrapper.find(“.notes”);
expect(notes.vnode.children.length).toEqual(2);
});
/** @test */
Unite.test(“It sees the form to create/edit notes”, () => {
let form = wrapper.find(“.form”);
expect(form.find(“input[name=id]”).exists()).toBe(true);
expect(form.find(“input[name=title]”).exists()).toBe(true);
expect(form.find(“textarea[name=body]”).exists()).toBe(true);
expect(form.find(“button”).exists()).toBe(true);
});
/** @test */
Unite.test(“It adds a new note when the button is clicked”, () => {
type(“input[name=title]”, “New Note”);
type(“textarea[name=body]”, “Note’s body”);
expect(wrapper.find(“.notes”).isEmpty()).toBe(true);
wrapper.find(“button”).trigger(“click”);
expect(wrapper.find(“.notes”).vnode.children.length).toEqual(1);
expect(wrapper.find(“.note:first-child”).text()).toEqual(“New Note”);
});
/** @helper It adds a new Note */
let addNote = (title, body) => {
let noteKey = wrapper.vm.notes.length;
wrapper.vm.notes.push({
id: (new Date).getTime(),
title: title,
body: body
});
wrapper.update();
return wrapper.vm.notes[noteKey];
};
/** @helper It types into an input */
let type = (selector, value) => {
let node = wrapper.find(selector);
node.element.value = value;
node.trigger(“input”);
};
</tests>
```
### When a note is clicked, we should see it in the form
So, given we have a note, when we click on the `.notes` child element, we should see the note data on the `.form` inputs. Let’s write the test:
```vue
// src/annotate.tests.vue
<tests>
// Other tests…
/** @test */
Unite.test(“It shows the clicked note on the form”, () => {
let note = addNote(“Note 1”, “Note’s body”);
wrapper.find(“.notes:first-child”).trigger(“click”);
let form = wrapper.find(“.form”);
expect(form.find(“input[name=id]”).element.value).toEqual(note.id);
expect(form.find(“input[name=title]”).element.value).toEqual(note.title);
expect(form.find(“textarea[name=title]”).element.value).toEqual(note.body);
});
</tests>
```
+ `@test: It shows the clicked note on the form`:
+ `let note = addNote(“Note 1”, “Note’s body”);`: It adds a new note using the `addNote` helper
+ `wrapper.find(“.notes:first-child”).trigger(“click”);`: It finds the `.notes` DOM element’s first child and triggers the click event
+ `let form = wrapper.find(“.form”);`: It finds the `.form` DOM element
+ `expect(form.find(“input[name=id]”).element.value).toEqual(String(note.id));`: It expects that the value of the input with the name `id` should be equal to the added note’s id (the `id` will be converted to a string)
+ `expect(form.find(“input[name=title]”).element.value).toEqual(note.title);`: It expects that the value of the input with the name `title` should be equal to the added note’s title
+ `expect(form.find(“textarea[name=title]”).element.value).toEqual(note.body);`: It expects that the value of the input with the name `body` should be equal to the added note’s body
Let’s fire off the tests:
1. `npm test`:
```
1) annotate.tests.vue @ It shows the clicked note on the form
expect(received).toEqual(expected)
Expected value to equal:
1508775148045
Received:
“”
```
This error is showing us that, after we clicked on the `.note` element, the input with the name `id` does not have the expected value. Right now, when we click on the `.note` element, nothing is being done. So let’s trigger an `edit` method when the `.note` element is clicked:
```vue
// src/annotate.tests.vue
<template>
<div class=”annotate”>
<ul class=”notes”>
<li class=”note” v-for=”note in notes” :key=”note.id” v-text=”note.title” @click=”edit(note)”></li>
</ul>
<div class=”form”>
<input type=”hidden” name=”id” v-model=”note.id”>
<input type=”text” name=”title” v-model=”note.title”>
<textarea name=”body” v-model=”note.body”></textarea>
<button @click=”save”>Save</button>
</div>
</div>
</template>
```
And now we should create the `edit` method:
```vue
// src/annotate.tests.vue
<script>
export default {
data() {
return {
note: {
id: “”,
title: “”,
body: “”
},
notes: []
};
},
methods: {
save() {
this.note.id = (new Date).getTime();
this.notes.push(this.note);
},
edit(note) {
this.note = note;
}
}
}
</script>
```
+ `@methods: edit(note)`:
+ `this.note = note;`: It sets the `note` object to the given note
2. `npm test`:
Now we get green! Our component should look like:
```vue
// src/annotate.tests.vue
<template>
<div class=”annotate”>
<ul class=”notes”>
<li class=”note” v-for=”note in notes” :key=”note.id” v-text=”note.title” @click=”edit(note)”></li>
</ul>
<div class=”form”>
<input type=”hidden” name=”id” v-model=”note.id”>
<input type=”text” name=”title” v-model=”note.title”>
<textarea name=”body” v-model=”note.body”></textarea>
<button @click=”save”>Save</button>
</div>
</div>
</template>
<script>
export default {
data() {
return {
note: {
id: “”,
title: “”,
body: “”
},
notes: []
};
},
methods: {
save() {
this.note.id = (new Date).getTime();
this.notes.push(this.note);
},
edit(note) {
this.note = note;
},
}
}
</script>
<tests>
let wrapper;
/**
* It mounts the component before each test
*/
Unite.beforeEachTest(() => {
wrapper = Unite.$mount($component);
});
/** @test */
Unite.test(“It displays all the notes”, () => {
addNote(“Note 1”, “Nody body”);
addNote(“Note 2”, “Nody body”);
let notes = wrapper.find(“.notes”);
expect(notes.vnode.children.length).toEqual(2);
});
/** @test */
Unite.test(“It sees the form to create/edit notes”, () => {
let form = wrapper.find(“.form”);
expect(form.find(“input[name=id]”).exists()).toBe(true);
expect(form.find(“input[name=title]”).exists()).toBe(true);
expect(form.find(“textarea[name=body]”).exists()).toBe(true);
expect(form.find(“button”).exists()).toBe(true);
});
/** @test */
Unite.test(“It adds a new note when the button is clicked”, () => {
type(“input[name=title]”, “New Note”);
type(“textarea[name=body]”, “Note’s body”);
expect(wrapper.find(“.notes”).isEmpty()).toBe(true);
wrapper.find(“button”).trigger(“click”);
expect(wrapper.find(“.notes”).vnode.children.length).toEqual(1);
expect(wrapper.find(“.note:first-child”).text()).toEqual(“New Note”);
});
/** @test */
Unite.test(“It shows the clicked note on the form”, () => {
let note = addNote(“Note 1”, “Note’s body”);
wrapper.find(“.note:first-child”).trigger(“click”);
let form = wrapper.find(“.form”);
expect(form.find(“input[name=id]”).element.value).toEqual(String(note.id));
expect(form.find(“input[name=title]”).element.value).toEqual(note.title);
expect(form.find(“textarea[name=body]”).element.value).toEqual(note.body);
});
/** @helper It adds a new Note */
let addNote = (title, body) => {
let noteKey = wrapper.vm.notes.length;
wrapper.vm.notes.push({
id: (new Date).getTime(),
title: title,
body: body
});
wrapper.update();
return wrapper.vm.notes[noteKey];
};
/** @helper It types into an input */
let type = (selector, value) => {
let node = wrapper.find(selector);
node.element.value = value;
node.trigger(“input”);
};
</tests>
```
### When a note is edited and saved, it should update the respective note in the list
So, given we have a note and that we are editing that note, when we modify the inputs and click on the `button`, we expect that the `note` object will be updated with the new input value.
Let’s write the test:
```vue
// src/annotate.tests.vue
<tests>
// Other tests…
/** @test */
Unite.test(“It updates a note object with the input data”, () => {
addNote(“Note 1”, “Note’s body”);
addNote(“Note 2”, “Note’s body”);
addNote(“Note 3”, “Note’s body”);
wrapper.find(“.note:nth-child(2)”).trigger(“click”);
type(“input[name=title]”, “New Title”);
wrapper.find(“button”).trigger(“click”);
expect(wrapper.vm.notes.length).toEqual(3);
expect(wrapper.vm.notes[1].title).toEqual(“New Title”);
});
</tests>
```
+ `@test: It updates a note object with the input data`:
+ `addNote(“Note 1”, “Note’s body”);`: It adds a new `note` object to the `notes` list
+ `addNote(“Note 2”, “Note’s body”);`: It adds a new `note` object to the `notes` list
+ `addNote(“Note 3”, “Note’s body”);`: It adds a new `note` object to the `notes` list
+ `wrapper.find(“.note:nth-child(2)”).trigger(“click”);`: It finds the second `.note` DOM element and triggers a click event
+ `type(“input[name=title]”, “New Title”);`: It sets the title form input to “New Title”
+ `wrapper.find(“button”).trigger(“click”);`: It triggers the click event on the `button` element
+ `expect(wrapper.vm.notes.length).toEqual(3);`: It asserts that the number of notes is 3
+ `expect(wrapper.vm.notes[1].title).toEqual(“New Title”);`: It asserts that the title of the second note is “New Title”
1. `npm test`:
```
1) annotate.tests.vue @ It updates a note object with the input data
expect(received).toEqual(expected)
Expected value to equal:
3
Received:
4
```
This shows that, instead of updating the second note, we are adding a new `note` object to the list! To correct that, let’s update the `save` function:
```vue
// src/annotate.tests.vue
<script>
export default {
…
methods: {
save() {
let note = this.note;
if(note.id !== “”) {
this.notes = this.notes.map(item => {
if(item.id === note.id) {
return note;
}
return item;
});
} else {
note.id = (new Date).getTime();
this.notes.push(note);
}
},
edit(note) {
this.note = note;
},
}
}
</script>
```
Let’s go through the new code:
+ `@methods: save`:
+ `let note = this.note;`: It gets the form values
+ `if(note.id !== “”)`: If the note’s `id` is not empty
+ `this.notes = this.notes.map()`: It maps through the `notes` list items
+ `if(item.id === note.id)`: If the item’s `id` is equal to the note’s `id`
+ `return note;`: It returns the note
+ `return item;`: Else, it returns the item
+ `else`: If the note `id` is empty
+ `note.id = (new Date).getTime();`: It sets a unique `id`
+ `this.notes.push(note);`: It pushed the new `note` object to the `notes` list
2. `npm test`:
Now we get green!
```vue
// src/annotate.tests.vue
<template>
<div class=”annotate”>
<ul class=”notes”>
<li class=”note” v-for=”noteItem in notes” :key=”noteItem.id” v-text=”noteItem.title” @click=”edit(noteItem)”></li>
</ul>
<div class=”form”>
<input type=”hidden” name=”id” v-model=”note.id”>
<input type=”text” name=”title” v-model=”note.title”>
<textarea name=”body” v-model=”note.body”></textarea>
<button @click=”save”>Save</button>
</div>
</div>
</template>
<script>
export default {
data() {
return {
note: {
id: “”,
title: “”,
body: “”
},
notes: []
};
},
methods: {
save() {
let note = this.note;
if(note.id !== “”) {
this.notes = this.notes.map(item => {
if(item.id === note.id) {
return note;
}
return item;
});
} else {
note.id = (new Date).getTime();
this.notes.push(note);
}
},
edit(note) {
this.note = note;
},
}
}
</script>
<tests>
let wrapper;
/**
* It mounts the component before each test
*/
Unite.beforeEachTest(() => {
wrapper = Unite.$mount($component);
});
/** @test */
Unite.test(“It displays all the notes”, () => {
addNote(“Note 1”, “Nody body”);
addNote(“Note 2”, “Nody body”);
let notes = wrapper.find(“.notes”);
expect(notes.vnode.children.length).toEqual(2);
});
/** @test */
Unite.test(“It sees the form to create/edit notes”, () => {
let form = wrapper.find(“.form”);
expect(form.find(“input[name=id]”).exists()).toBe(true);
expect(form.find(“input[name=title]”).exists()).toBe(true);
expect(form.find(“textarea[name=body]”).exists()).toBe(true);
expect(form.find(“button”).exists()).toBe(true);
});
/** @test */
Unite.test(“It adds a new note when the button is clicked”, () => {
type(“input[name=title]”, “New Note”);
type(“textarea[name=body]”, “Note’s body”);
expect(wrapper.find(“.notes”).isEmpty()).toBe(true);
wrapper.find(“button”).trigger(“click”);
expect(wrapper.find(“.notes”).vnode.children.length).toEqual(1);
expect(wrapper.find(“.note:first-child”).text()).toEqual(“New Note”);
});
/** @test */
Unite.test(“It shows the clicked note on the form”, () => {
let note = addNote(“Note 1”, “Note’s body”);
wrapper.find(“.note:first-child”).trigger(“click”);
let form = wrapper.find(“.form”);
expect(form.find(“input[name=id]”).element.value).toEqual(String(note.id));
expect(form.find(“input[name=title]”).element.value).toEqual(note.title);
expect(form.find(“textarea[name=body]”).element.value).toEqual(note.body);
});
/** @test */
Unite.test(“It updates a note object with the input data”, () => {
addNote(“Note 1”, “Note’s body”);
addNote(“Note 2”, “Note’s body”);
addNote(“Note 3”, “Note’s body”);
wrapper.find(“.note:nth-child(2)”).trigger(“click”);
type(“input[name=title]”, “New Title”);
wrapper.find(“button”).trigger(“click”);
expect(wrapper.vm.notes.length).toEqual(3);
expect(wrapper.vm.notes[1].title).toEqual(“New Title”);
});
/** @helper It adds a new Note */
let addNote = (title, body) => {
let noteKey = wrapper.vm.notes.length;
wrapper.vm.notes.push({
id: (new Date).getTime(),
title: title,
body: body
});
wrapper.update();
return wrapper.vm.notes[noteKey];
};
/** @helper It types into an input */
let type = (selector, value) => {
let node = wrapper.find(selector);
node.element.value = value;
node.trigger(“input”);
};
</tests>
```
### When a `new` button is clicked, we should see a blank form
Finally, we need a button to create a new note. When we click it, we clear out the form so we can start the new note.
Let’s get started with the test:
```vue
// src/annotate.tests.vue
<tests>
// Other tests…
/** @test */
Unite.test(“It shows the blank form when the new button is clicked”, () => {
addNote(“Note Title”, “Note’s body”);
wrapper.find(“.note:first-child”).trigger(“click”);
expect(wrapper.find(“input[name=title]”).element.value).toEqual(“Note Title”);
wrapper.find(“.new-note”).trigger(“click”);
expect(wrapper.find(“input[name=id]”).element.value).toEqual(“”);
expect(wrapper.find(“input[name=title]”).element.value).toEqual(“”);
expect(wrapper.find(“textarea[name=body]”).element.value).toEqual(“”);
});
</tests>
```
+ `@test: It shows the blank form when the new button is clicked`:
+ `addNote(“Note Title”, “Note’s body”);`: It adds a new `note` with the `addNote` helper function
+ `wrapper.find(“.note:first-child”).trigger(“click”);`: It finds the first `.note` DOM element and triggers a click event
+ `expect(wrapper.find(“input[name=title]”).element.value).toEqual(“Note Title”);`: It expects that the `title` input’s value is the note’s value
+ `wrapper.find(“.new-note”).trigger(“click”);`: It triggers the click event on the `.new-note` element
+ `expect(wrapper.find(“input[name=id]”).element.value).toEqual(“”);`: It expects that the `id` input’s value to be empty
+ `expect(wrapper.find(“input[name=title]”).element.value).toEqual(“”);`: It expects that the `title` input’s value to be empty
+ `expect(wrapper.find(“textarea[name=body]”).element.value).toEqual(“”);`: It expects that the `body` input’s value to be empty
Let’s start testing:
1. `npm test`:
```
1) annotate.tests.vue @ It shows the blank form when the new button is clicked
Error: [vue-test-utils]: find did not return .new-note, cannot call trigger() on empty Wrapper
```
It’s trying to find the `.new-note` element and trigger the `click` event, but it doesn’t exist! Let’s create it:
```vue
// ./annotate.tests.vue
<template>
<div class=”annotate”>
<div class=”new-note”>Add new note</div>
<ul class=”notes”>
<li class=”note” v-for=”noteItem in notes” :key=”noteItem.id” v-text=”noteItem.title” @click=”edit(noteItem)”></li>
</ul>
<div class=”form”>
<input type=”hidden” name=”id” v-model=”note.id”>
<input type=”text” name=”title” v-model=”note.title”>
<textarea name=”body” v-model=”note.body”></textarea>
<button @click=”save”>Save</button>
</div>
</div>
</template>
```
2. `npm test`:
```
1) annotate.tests.vue @ It shows the blank form when the new button is clicked
expect(received).toEqual(expected)
Expected value to equal:
“”
Received:
“1508781636415”
```
Once it clicked the `.new-button`, it expected the form input to be cleared. But currently, it’s not doing anything. Let’s trigger the `newNote` method:
```vue
// ./annotate.tests.vue
<template>
<div class=”annotate”>
<div class=”new-note” @click=”newNote”>Add new note</div>
<ul class=”notes”>
<li class=”note” v-for=”noteItem in notes” :key=”noteItem.id” v-text=”noteItem.title” @click=”edit(noteItem)”></li>
</ul>
<div class=”form”>
<input type=”hidden” name=”id” v-model=”note.id”>
<input type=”text” name=”title” v-model=”note.title”>
<textarea name=”body” v-model=”note.body”></textarea>
<button @click=”save”>Save</button>
</div>
</div>
</template>
```
Now we should create the `newNote` method:
```vue
// ./annotate.tests.vue
<script>
export default {
…
methods: {
…
newNote() {
this.note = {
id: “”,
title: “”,
body: “”
};
}
}
}
</script>
```
+ `@methods: newNote`:
+ `this.note = { id: “”, title: “”, body: “” };`: It sets the `note` variable to an empty `note` object
3. `npm test`:
And we are at green! So, our final `annotate.tests.vue` component should look something like:
```vue
// ./annotate.tests.vue
<template>
<div class=”annotate”>
<div class=”new-note” @click=”newNote”>Add new note</div>
<ul class=”notes”>
<li class=”note” v-for=”noteItem in notes” :key=”noteItem.id” v-text=”noteItem.title” @click=”edit(noteItem)”></li>
</ul>
<div class=”form”>
<input type=”hidden” name=”id” v-model=”note.id”>
<input type=”text” name=”title” v-model=”note.title”>
<textarea name=”body” v-model=”note.body”></textarea>
<button @click=”save”>Save</button>
</div>
</div>
</template>
<script>
export default {
data() {
return {
note: {
id: “”,
title: “”,
body: “”
},
notes: []
};
},
methods: {
save() {
let note = this.note;
if(note.id !== “”) {
this.notes = this.notes.map(item => {
if(item.id === note.id) {
return note;
}
return item;
});
} else {
note.id = (new Date).getTime();
this.notes.push(note);
}
},
edit(note) {
this.note = note;
},
newNote() {
this.note = {
id: “”,
title: “”,
body: “”
};
}
}
}
</script>
<tests>
let wrapper;
/**
* It mounts the component before each test
*/
Unite.beforeEachTest(() => {
wrapper = Unite.$mount($component);
});
/** @test */
Unite.test(“It displays all the notes”, () => {
addNote(“Note 1”, “Nody body”);
addNote(“Note 2”, “Nody body”);
let notes = wrapper.find(“.notes”);
expect(notes.vnode.children.length).toEqual(2);
});
/** @test */
Unite.test(“It sees the form to create/edit notes”, () => {
let form = wrapper.find(“.form”);
expect(form.find(“input[name=id]”).exists()).toBe(true);
expect(form.find(“input[name=title]”).exists()).toBe(true);
expect(form.find(“textarea[name=body]”).exists()).toBe(true);
expect(form.find(“button”).exists()).toBe(true);
});
/** @test */
Unite.test(“It adds a new note when the button is clicked”, () => {
type(“input[name=title]”, “New Note”);
type(“textarea[name=body]”, “Note’s body”);
expect(wrapper.find(“.notes”).isEmpty()).toBe(true);
wrapper.find(“button”).trigger(“click”);
expect(wrapper.find(“.notes”).vnode.children.length).toEqual(1);
expect(wrapper.find(“.note:first-child”).text()).toEqual(“New Note”);
});
/** @test */
Unite.test(“It shows the clicked note on the form”, () => {
let note = addNote(“Note 1”, “Note’s body”);
wrapper.find(“.note:first-child”).trigger(“click”);
let form = wrapper.find(“.form”);
expect(form.find(“input[name=id]”).element.value).toEqual(String(note.id));
expect(form.find(“input[name=title]”).element.value).toEqual(note.title);
expect(form.find(“textarea[name=body]”).element.value).toEqual(note.body);
});
/** @test */
Unite.test(“It updates a note object with the input data”, () => {
addNote(“Note 1”, “Note’s body”);
addNote(“Note 2”, “Note’s body”);
addNote(“Note 3”, “Note’s body”);
wrapper.find(“.note:nth-child(2)”).trigger(“click”);
type(“input[name=title]”, “New Title”);
wrapper.find(“button”).trigger(“click”);
expect(wrapper.vm.notes.length).toEqual(3);
expect(wrapper.vm.notes[1].title).toEqual(“New Title”);
});
/** @test */
Unite.test(“It shows the blank form when the new button is clicked”, () => {
addNote(“Note Title”, “Note’s body”);
wrapper.find(“.note:first-child”).trigger(“click”);
expect(wrapper.find(“input[name=title]”).element.value).toEqual(“Note Title”);
wrapper.find(“.new-note”).trigger(“click”);
expect(wrapper.find(“input[name=id]”).element.value).toEqual(“”);
expect(wrapper.find(“input[name=title]”).element.value).toEqual(“”);
expect(wrapper.find(“textarea[name=body]”).element.value).toEqual(“”);
});
/** @helper It adds a new Note */
let addNote = (title, body) => {
let noteKey = wrapper.vm.notes.length;
wrapper.vm.notes.push({
id: (new Date).getTime(),
title: title,
body: body
});
wrapper.update();
return wrapper.vm.notes[noteKey];
};
/** @helper It types into an input */
let type = (selector, value) => {
let node = wrapper.find(selector);
node.element.value = value;
node.trigger(“input”);
};
</tests>
```
## Conclusions
In this example, we created a very simple note-taking `Vue.js` component using TDD with `unite.js`. The source code can be found here!
From here, there are two things that need to be done:
+ We did not worry about styling our component. So we need to make it pretty!
+ On this example, we focused on the front-end and the notes are not persisted. To to this, you should modify the save method to send an ajax request to the back-end where you can persist the the data.
So show off your own Annotate component below! Also, if you have any questions or ideas to make our Annotate component better, leave your comments below!
If you enjoyed this show your support and tell your friends!
Thank you!