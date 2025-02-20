---
title: Your First App
---

### The Goal

Sometimes it can be easier to learn something if you have a goal to work towards. In this section we will build a Todo app with Svelte Native. It will have the following functionality:

* Basic design
  * Two-tab layout
  * One tab shows active tasks and lets you add new tasks
  * Second tab lists completed tasks
* Basic functionality
  * Add tasks: Users can add tasks as text
  * View tasks: Newly added tasks are listed as active and can be tapped
  * Complete tasks: Tapping an active task shows an action dialog with options
  * Delete tasks: Tapping an active or completed task shows an action dialog with options
* Advanced design
  * Input and button for adding tasks are styled
  * Tabs are styled
  * Active tasks are styled
  * Completed tasks are styled

![TodoApp](/media/todoapp/nativescript-svelte-todo2.gif)


### Prerequisites

Before you start, please ensure you have at least followed the [Quick Start Guide](docs#quick-start) and can get an application to run on your mobile device or emulator.

This guide assumes a existing familiarity with the Svelte framework. Run through [Svelte's excellent tutorial](https://svelte.dev/tutorial/basics) to get up to speed.

### Basic Design

We will start our from a fresh app template:

```bash
$ degit halfnelson/svelte-native-template todoapp
$ cd todoapp
$ npm install
```

Remove the default `.btn` rule from `app.css` and set the contents of App.svelte to:

```html
<!--{ filename: 'App.svelte' }-->
<page class="page">
    <actionBar title="My Tasks" class="action-bar" />
    
    <tabView androidTabsPosition="bottom">

      <tabViewItem title="To Do" textWrap="true">
        <label>
            This tab will list active tasks and will let users add new tasks.
        </label>
      </tabViewItem>
      
      <tabViewItem title="Completed">
        <label text="This tab will list completed tasks for tracking." textWrap="true" />
      </tabViewItem>

    </tabView>
</page>
```
 > **NOTE** Notice that all tags start with a lower case letter. This is different to other NativeScript implementations. The lower case letter lets the Svelte compiler know that these are NativeScript views and not Svelte components. Think of `<page>` and `<actionBar>` as just another set of application building blocks like `<ul>` and `<div>`.

#### What's all that then?

The `<page>` element is the top-level user interface element of every Svelte-Native app. All other user interface elements are nested within.

The `<actionBar>` element shows an action bar for the `<page>`. A `<page>` cannot contain more than one `<actionBar>`.

Typically, after the `<actionBar>`, you will have navigation components (such as a drawer or a tab view) or layout components. These elements control the layout of your app and let you determine how to place other user interface elements inside.

The class names on `page` and `actionBar` are to help with styling, the default stylesheet includes a reference to the NativeScript core theme. More information on the core theme can be found in the [Nativescript Docs](https://docs.nativescript.org/ui/theme)

The `<label>` tags have been used differently. One has the `text=` attribute, while the other has the text between the opening and closing tags. Plain text between tags will be automatically assigned to the `text` attribute of the tag.

#### Progress So Far

<img src="/media/todoapp/todo-basic-design-1.png" alt="tab 1" width=300> <img src="/media/todoapp/todo-basic-design-2.png" alt="tab 2" width=300>


### Basic Functionality: Add Tasks

We have our basic design, lets allow the user to add some tasks.

Replace the contents of the first `<tabViewItem>` with:

```html
<gridLayout columns="2*,*" rows="*, 3*">
    <!-- Configures the text field and ensures that pressing Return on the keyboard 
        produces the same result as tapping the button. -->
    <textField col="0" row="0" bind:text="{textFieldValue}" hint="Type new task..." editable="true"
        on:returnPress="{onButtonTap}" />
    <button col="1" row="0" text="Add task" on:tap="{onButtonTap}" />

    <listView class="list-group" items="{todos}" on:itemTap="{onItemTap}" row="1" colSpan="2">
        <Template let:item>
            <label text="{item.name}" class="list-group-item-heading" textWrap="true" />
        </Template>
    </listView>
</gridLayout>
```

and to the bottom of the file add a script tag:
```html
<script>
    import { Template } from 'svelte-native/components'
    
    let todos = []
    let textFieldValue = ""

    function onItemTap(args) {
      console.log('Item with index: ' + args.index + ' tapped');
    }

    function onButtonTap() {
      if (textFieldValue === "") return; // Prevents users from entering an empty string.
      console.log("New task added: " + textFieldValue + "."); // Logs the newly added task in the console for debugging.
      todos = [{ name: textFieldValue }, ...todos] // Adds tasks in the ToDo array. Newly added tasks are immediately shown on the screen.
      textFieldValue = ""; // Clears the text field so that users can start adding new tasks immediately.
    }
</script>
```

#### What did we just do?

To allow the user to enter a todo item, we need to capture the name of the task. We did this by adding a `<textField>`. A `<button>` was added to submit the task and a `<listView>` to display the task.

Since this functionality required adding 3 elements to the tabview, we use layouts to tell NativeScript where to place each item. `<stackLayout>` places items in a row either vertical or horizontal. We used it to place our input form above the `<listView>`. `<gridView>` is used to layout items in a predefined grid. It is used to place the `<button>` to the right and take up half the space of the `<textInput>`.

The `<listView>` contains a `<Template>` which is a Svelte component used to render each item. The template component needs to be imported just like all Svelte components.

When `onButtonTap` callback is fired, the code we added to the script element, will build a new `todos` array including the added item, and clear the text field. The `onItemTap` callback will just log which list item index was tapped using `console.log` which works fine in NativeScript.

> **NOTE** `<listView>` will look for the first `<Template>` component in its children. The template component acts similar to a slot and will render its content for each item. This is exposed to the content as `item` via the `let:item` on the template element.

#### Progress So Far

<img alt="We can add items" src="/media/todoapp/todo-add-item.png" width=350>

It isn't pretty, but it works!

### Basic functionality: Complete/Delete Tasks

Nobody likes a todo list that only gets longer. We should add the ability to mark a task as complete, or to remove it if we added it by accident.

At the very top of the script tag, add this import
```js
  import { action } from "tns-core-modules/ui/dialogs";
```

Near the top of the script tag after the `let todos=[]` add another declaration `let dones=[]` so completed tasks have somewhere to go.


Then replace our `onItemTap` function with this new one:

```js
  function onItemTap(args) {
    action("What do you want to do with this task?", "Cancel", [
      "Mark completed",
      "Delete forever"
    ]).then(result => {
      console.log(result); // Logs the selected option for debugging.
      let item = todos[args.index];
      switch (result) {
        case "Mark completed":
          dones = [item, ...dones]; // Places the tapped active task at the top of the completed tasks.
          todos = todos.filter(t => t != item); // Removes the tapped active  task.
          break;
        case "Delete forever":
          todos = todos.filter(t => t != item); // Removes the tapped active task.
          break;
        case "Cancel" || undefined: // Dismisses the dialog
          break;
      }
    });
  }
```

#### Breaking it down

Native script comes with a `dialogs` module that allows us to show small modal windows to obtain data from a user. We added an import to this module so that we could use the `action` method we added to the `onItemTap` method. When the user selects "Mark completed" we find the item using the `args.index` we get from the event, and remove the item from the `todos`, then we add the item to our new `dones` array. The delete command just removes the item from the `todos`.

> **NOTE** Notice that we reassign the `dones` and `todos` variables during delete or complete operations. Svelte's reactive variables work at the top level and cannot detect changes in an array. By assigning a new value to `dones` and `todos` we are ensuring that any template that depends on those variables will be updated.

#### Progress So Far

<img alt="Popup In Action" src="/media/todoapp/todo-mark-complete.png" width=350>



### Basic functionality: The Completed Tab

To get that sense of satisfaction from completing an item on your todo list, it would be good to be able to see the item on the completed tab. In this section we will add a listview to display the items and allow you to delete them or restore them to the todos using an action.

First add the `listView` below to the second tab replacing the `label`
```html
<listView class="list-group" items="{dones}" on:itemTap="{onDoneTap}">
	<Template let:item>
		<label text="{item.name}" class="list-group-item-heading" textWrap="true" />
	</Template>
</listView>
```

Then add the code for the onDoneTap to the script block:

```js
function onDoneTap(args) {
  action("What do you want to do with this task?", "Cancel", [
    "Mark To Do",
    "Delete forever"
  ]).then(result => {
    console.log(result); // Logs the selected option for debugging.
    let item = dones[args.index]
    switch (result) {
      case "Mark To Do":
        todos = [item, ...todos]; // Places the tapped active task at the top of the completed tasks.
        dones = dones.filter(t => t != item); // Removes the tapped active  task.
        break;
      case "Delete forever":
        dones = dones.filter(t => t != item); // Removes the tapped active task.
        break;
      case "Cancel" || undefined: // Dismisses the dialog
        break;
    }
  });
}
```

#### What we just did

To display our done items we added the `listView` to the "completed" `tabViewItem` and bound it to the `dones` variable we defined in last step.

We added an event handler to handle taps on the "completed" items. This handler is very similar to the handler added in the last section, except that it works on the `dones` array and not the `todos`.

### Advanced design: Styled input 

The basic functionality of the todo app is complete. But it isn't going to win any beauty contests. To delight our users we will take a few minutes to apply some styling. In this section we will style the text box and button elements.

At the bottom of `App.svelte` add the following style tag:

```html
<style>
  button { 
      font-size: 15; 
      font-weight: bold; 
      color: white; 
      background-color: #2847D2; 
      height: 40;
      margin-top: 10; 
      margin-bottom: 10; 
      margin-right: 10; 
      margin-left: 10; 
      border-radius: 20px; 
  }

  textField {
      font-size: 20;
      color: #2847D2;
      margin-top: 10;
      margin-bottom: 10;
      margin-right: 5;
      margin-left: 20;
  }
</style>
```

#### A style tag in a native application!?

When you work with NativeScript and Svelte, you can use application-wide CSS, scoped CSS, or inline CSS to style your app. Application-wide CSS is applied first and is handled in `app.css` in the root of your project. This tutorial does not explore application-wide CSS. See also: [Styling](https://docs.nativescript.org/ui/styling).

Scoped CSS is applied to the current component only and is handled in each component's `<style>` block. This tutorial relies almost exclusively on scoped CSS and inline CSS. See also: [Scoped Styles](https://v3.svelte.technology/docs#scoped-styles).

With type selectors, you can select a UI component and apply styling to it. To select a type, use the component name as provided in the code. For example, to select the tab view, use `TabView`.

#### Progress So Far

<img alt="style button" src="/media/todoapp/todo-styled-button.png" width=350>


### Advanced design: Styled tabs

The application is looking better, but we can do something about those tabs.

Add the `selectedTabTextColor` and `tabTextFontSize` property to the `<TabView>`.

```html
  <tabView androidTabsPosition="bottom" selectedTabTextColor="#2847D2" tabTextFontSize="15" >
```

Apply the `textTransform` property to the separate tabs. You can use this property only on the `<TabViewItem>` level.

```html
  <tabViewItem title="To Do" textTransform="uppercase" >
```

```html
  <tabViewItem title="Completed" textTransform="uppercase">
```

#### That is not CSS!

`<TabView>` provides some styling properties out of the box. You can apply a text transform to each tab title (`textTransform`) and change the font size and color globally (`tabTextFontSize`, `tabTextColor`, `selectedTabTextColor`). You can also change the background color of your tabs `tabBackgroundColor`.

> **TIP** Most css properties in NativeScript have a corresponding attribute you can apply directly to the element

#### Progress So Far

<img alt="style button" src="/media/todoapp/todo-styled-tabs.png" width=350>


### Advanced design: Styled Lists

The list items are a little bunched up. Lets add some padding to them, and make the active todo items coloured and the completed items crossed out.

Edit the two `<listView>` tags and add the `separatorColor="transparent"` as an attribute:

```html
<listView class="list-group" items="{todos}" on:itemTap="{onItemTap}"  separatorColor="transparent">
```
and 
```html
<listView class="list-group" items="{dones}" on:itemTap="{onDoneTap}" row="1" colSpan="2" separatorColor="transparent">
```

To the label in the `listView` for the `todos` add `todo-item active` to the class attribute
```html
 <label text="{item.name}" class="list-group-item-heading todo-item active" textWrap="true" />
```
to the dones label add `todo-item completed`
```html
 <label text="{item.name}" class="list-group-item-heading todo-item completed" textWrap="true" />
```

Add the following CSS rules to the `style` tag

```css

.todo-item {
  font-size: 20;
  margin-left: 20;
  padding-top: 5;
  padding-bottom: 10;
}

.todo-item.active {
  font-weight: bold;
  color: #2847D2;
}

.todo-item.completed {
  color: #d3d3d3;
  text-decoration: line-through;
}

```

#### I see what you did there

In NativeScript you aren't restricted to just using element names as CSS selectors. We added some classes to the labels and applied CSS rules to those classes.
We also applied a `separatorColor` attribute directly to `listView` to remove the lines between the items for a cleaner list.

#### Our Finished Product

<img alt="todos" src="/media/todoapp/todo-styled-list1.png" width=350>
<img alt="dones" src="/media/todoapp/todo-styled-list2.png" width=350>

