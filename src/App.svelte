<script>
  import { onMount, onDestroy } from "svelte";
  import { Editor } from "@tiptap/core";
  import StarterKit from "@tiptap/starter-kit";
  import { Image as ImageTippap } from "@tiptap/extension-image";
  import Dropcursor from "@tiptap/extension-dropcursor";
  import Code from "@tiptap/extension-code";
  import Link from "@tiptap/extension-link";
  import Modal, { getModal } from "./Modal.svelte";
  import { Button, TabContent, TabPane } from "sveltestrap";
  import Resizer from "react-image-file-resizer";
  import axios from "axios";

  let element;
  let editor;
  let myImage;

  onMount(() => {
    editor = new Editor({
      element: element,
      extensions: [
        StarterKit,
        ImageTippap,
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

  async function uploadImage() {
    const fileInput = document.querySelector("#myImage");
    const formData = new FormData();

    formData.append("myImage", fileInput.files[0]);
    console.log(fileInput.files[0]);

    const res = await fetch("http://localhost:2002/", {
      method: "POST",
      // headers: new Headers({
      //   "Content-Type": "undefined",
      // }),
      body: formData,
    });

    res
      ?.json()
      .then((result) => {
        console.log(result.filename);

        const urlImage = "http://localhost:2002/" + result.filename;

        editor.chain().focus().setImage({ src: urlImage }).run();
      })
      .catch((error) => {
        alert(error);
      });
  }

  const resize = Resizer.imageFileResizer;
  let rawImgs;

  let resizeImage = async (file) => {
    let _URL = window.URL || window.webkitURL;
    let img = new Image();

    let width = 0;
    let height = 0;

    img.src = _URL.createObjectURL(file);

    console.log("natural", img.clientHeight);

    img.onload = function () {
      width = img.width;
      height = img.height;

      console.log("ancho: " + this.width + " " + "alto: " + height);
    }; //onload

    // console.log("ancho_fuera", width);

    // return new Promise((resolve, reject) => {
    //   resize(file, 1200, 500, "JPEG", 100, 0, (uri) => resolve(uri), "blob");
    // });
    // return result;
  };

  let resizeImages = async (files) => {
    let resized = [];
    for (let i = 0; i < files.length; i++)
      resized.push(await resizeImage(files[i]));
    return resized;
  };

  let onSubmit = async () => {
    let formData = new FormData();

    console.log(rawImgs);

    let resizedImgs = await resizeImages(rawImgs);

    resizedImgs.forEach((resizedImg) => formData.append("myImage", resizedImg));

    let res = await axios.post("http://localhost:2002/", formData);
    console.log(res.data);
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
      <form>
        <input
          type="file"
          accept=".jpg, .jpeg"
          on:change={(e) => onFileSelected(e)}
          bind:files={rawImgs}
          multiple
        />
        <Button primary type="button" on:click={onSubmit}>Upload</Button>
      </form>
    </TabPane>
  </TabContent>
</Modal>

<div bind:this={element} />
hello Dan Hi Ana

<div id="prueba" />

<style>
  button.active {
    background: black;
    color: white;
  }
</style>
