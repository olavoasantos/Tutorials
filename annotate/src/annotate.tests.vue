<template>
    <div class="annotate">
        <div class="new-note" @click="newNote">Add new note</div>
        <ul class="notes">
            <li class="note" v-for="noteItem in notes" :key="noteItem.id" v-text="noteItem.title" @click="edit(noteItem)"></li>
        </ul>
        <div class="form">
            <input type="hidden" name="id" v-model="note.id">
            <input type="text" name="title" v-model="note.title">
            <textarea name="body" v-model="note.body"></textarea>
            <button @click="save">Save</button>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                note: {
                    id: "",
                    title: "",
                    body: ""
                },
                notes: []
            };
        },
        methods: {
            save() {
                let note = this.note;
                if(note.id !== "") {
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
                    id: "",
                    title: "",
                    body: ""
                };
            }
        }
    }
</script>

<tests>
    let wrapper;

    /**
    *   It mounts the component before each test
    */
    Unite.beforeEachTest(() => {
        wrapper = Unite.$mount($component);
    });

    /** @test */
    Unite.test("It displays all the notes", () => {
        addNote("Note 1", "Nody body");
        addNote("Note 2", "Nody body");

        let notes = wrapper.find(".notes");
        expect(notes.vnode.children.length).toEqual(2);
    });

    /** @test */
    Unite.test("It sees the form to create/edit notes", () => {
        let form = wrapper.find(".form");
        
        expect(form.find("input[name=id]").exists()).toBe(true);
        expect(form.find("input[name=title]").exists()).toBe(true);
        expect(form.find("textarea[name=body]").exists()).toBe(true);
        expect(form.find("button").exists()).toBe(true);
    });

    /** @test */
    Unite.test("It adds a new note when the button is clicked", () => {
        type("input[name=title]", "New Note");
        type("textarea[name=body]", "Note's body");
        
        expect(wrapper.find(".notes").isEmpty()).toBe(true);

        wrapper.find("button").trigger("click");

        expect(wrapper.find(".notes").vnode.children.length).toEqual(1);
        expect(wrapper.find(".note:first-child").text()).toEqual("New Note");
    });

    /** @test */
    Unite.test("It shows the clicked note on the form", () => {
        let note = addNote("Note 1", "Note's body");

        wrapper.find(".note:first-child").trigger("click");
        
        let form = wrapper.find(".form");
        expect(form.find("input[name=id]").element.value).toEqual(String(note.id));
        expect(form.find("input[name=title]").element.value).toEqual(note.title);
        expect(form.find("textarea[name=body]").element.value).toEqual(note.body);
    });

    /** @test */
    Unite.test("It updates a note object with the input data", () => {
        addNote("Note 1", "Note's body");
        addNote("Note 2", "Note's body");
        addNote("Note 3", "Note's body");

        wrapper.find(".note:nth-child(2)").trigger("click");
        type("input[name=title]", "New Title");
        wrapper.find("button").trigger("click");

        expect(wrapper.vm.notes.length).toEqual(3);
        expect(wrapper.vm.notes[1].title).toEqual("New Title");
    });

    /** @test */
    Unite.test("It shows the blank form when the new button is clicked", () => {
        addNote("Note Title", "Note's body");
        wrapper.find(".note:first-child").trigger("click");

        expect(wrapper.find("input[name=title]").element.value).toEqual("Note Title");

        wrapper.find(".new-note").trigger("click");

        expect(wrapper.find("input[name=id]").element.value).toEqual("");
        expect(wrapper.find("input[name=title]").element.value).toEqual("");
        expect(wrapper.find("textarea[name=body]").element.value).toEqual("");
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
        node.trigger("input");
    };
</tests>