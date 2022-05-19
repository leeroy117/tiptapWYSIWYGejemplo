<script>
  import { onMount, onDestroy } from "svelte";
  import { Editor } from "@tiptap/core";
  import StarterKit from "@tiptap/starter-kit";
  import Image from "@tiptap/extension-image";
  import Dropcursor from "@tiptap/extension-dropcursor";
  import Code from "@tiptap/extension-code";
  import Link from "@tiptap/extension-link";
  import Modal, { getModal } from "./Modal.svelte";
  import { Button, TabContent, TabPane } from "sveltestrap";

  let element;
  let editor;
  let fileinput;

  onMount(() => {
    editor = new Editor({
      element: element,
      extensions: [
        StarterKit,
        Image,
        Dropcursor,
        Code,
        Link.configure({
          openOnClick: true,
          linkOnPaste: true,
        }),
      ],
      content: "<p>Hello World! 🌍️ </p>",
      onTransaction: () => {
        // force re-render so `editor.isActive` works as expected
        editor = editor;
      },
    });
  });

  onDestroy(() => {
    if (editor) {
      editor.destroy();
    }
  });

  const onFileSelected = (e) => {
    let image = e.target.files[0];
    let reader = new FileReader();
    reader.readAsDataURL(image);
    reader.onload = (e) => {
      console.log(reader);
      // avatar = e.target.result;
    };
  };

  const setLink = () => {
    const previousUrl = editor.getAttributes("link").href;
    const url = window.prompt("URL", previousUrl);

    if (url === null) {
      return;
    }

    // empty
    if (url === "") {
      editor.chain().focus().extendMarkRange("link").unsetLink().run();

      return;
    }

    editor.chain().focus().extendMarkRange("link").setLink({ href: url }).run();
  };

  const addImage = () => {
    const url = window.prompt("URL");

    if (url) {
      editor.chain().focus().setImage({ src: url }).run();
    }
  };
</script>

{#if editor}
  <button
    on:click={() => editor.chain().focus().toggleHeading({ level: 1 }).run()}
    class:active={editor.isActive("heading", { level: 1 })}
  >
    H1
  </button>
  <button
    on:click={() => editor.chain().focus().toggleHeading({ level: 2 }).run()}
    class:active={editor.isActive("heading", { level: 2 })}
  >
    H2
  </button>
  <button
    on:click={() => editor.chain().focus().toggleHeading({ level: 3 }).run()}
    class:active={editor.isActive("heading", { level: 3 })}
  >
    H3
  </button>
  <button
    on:click={() => editor.chain().focus().setParagraph().run()}
    class:active={editor.isActive("paragraph")}
  >
    P
  </button>

  <button
    on:click={() => editor.chain().focus().toggleBulletList().run()}
    class:active={editor.isActive("bulletList")}
  >
    bullet list
  </button>

  <button
    on:click={() => editor.chain().focus().toggleOrderedList().run()}
    class:active={editor.isActive("orderedList")}
  >
    ordered list
  </button>

  <button on:click={setLink} class:active={editor.isActive("link")}>
    setLink
  </button>

  <button
    on:click={() => editor.chain().focus().unsetLink().run()}
    disabled={!editor.isActive("link")}
  >
    unsetLink
  </button>

  <button on:click={() => getModal().open()}>add image</button>
{/if}

<Modal>
  <h1>Hello Leeroy!</h1>
  <TabContent>
    <TabPane tabId="link" tab="Link" active>
      <h2>Link</h2>
      <Button primary on:click={addImage}>Click to Copy</Button>
    </TabPane>
    <TabPane tabId="simpleUpload" tab="Upload (simple)">
      <h2>Upload (simple)</h2>
      <input
        type="file"
        accept=".jpg, .jpeg, .png"
        on:change={(e) => onFileSelected(e)}
        bind:this={fileinput}
      />
    </TabPane>
  </TabContent>
</Modal>

<div bind:this={element} />
hello Dan Hi Ana

<style>
  button.active {
    background: black;
    color: white;
  }
</style>
