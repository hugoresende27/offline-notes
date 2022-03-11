<template>
  <div class="flex w-screen h-screen text-gray-700">
    <div class="flex flex-col flex-shrink-0 w-64 border-r border-gray-300 bg-gray-100">
      <!-- Sidebar -->
    </div>
    <div class="flex flex-col flex-grow">
      <!-- Main Content -->
      <div class="flex flex-col flex-grow overflow-auto">
        <editor-content :editor="editor" />
      </div>
    </div>
  </div>
</template>


<script setup>
import { useEditor, EditorContent } from '@tiptap/vue-3'
import StarterKit from '@tiptap/starter-kit'

const editor = useEditor({
  content: 'Some Content',
  extensions: [
    StarterKit,
  ],
  editorProps: {
    attributes: {
      class: "prose my-6 mx-auto focus:outline-none"
    }
  }
})

const getDatabase = async () => {
  return new Promise((resolve, reject) => {
    let db = window.indexedDB.open('notes');

    db.onerror = () => {
      reject('Error opening the database.')
    }

    db.onsuccess = e => {
      console.log('db.onsuccess', e)
      resolve(e.target.result);
    }

    db.onupgradeneeded = e => {
      console.log('db.onupgraded', e)
      e.target.result.createObjectStore('notes')
    }
  });
}

getDatabase();

</script>

<style></style>