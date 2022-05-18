<script>
  import { onMount, onDestroy } from "svelte";
  import { Editor } from "@tiptap/core";
  import StarterKit from "@tiptap/starter-kit";
  import Image from "@tiptap/extension-image";
  import Dropcursor from "@tiptap/extension-dropcursor";
  import Code from "@tiptap/extension-code";
  import Link from "@tiptap/extension-link";

  let element;
  let editor;

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

  <button on:click={addImage}>add image from URL</button>
{/if}

<div bind:this={element} />
hello Dan Hi Ana

<style>
  button.active {
    background: black;
    color: white;
  }
</style>
