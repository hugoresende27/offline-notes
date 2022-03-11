<template>
  <div class="flex w-screen h-screen text-gray-700">
    <div class="flex flex-col flex-shrink-0 w-64 border-r border-gray-300 bg-gray-100">
          <!-- SIDE BAR ------------------------------ -->
          <div class="h-0 overflow-auto flex-grow">
            <div class="mt-4">
              <a class="flex items-center h-8 text-sm pl-8 pr-3"
                 href="#"
                 v-for="note in notes"
                 :key="note.created"
                 >

                 <span
                    class="ml-2 leading-none">

                    {{ new Date(note.created).toLocaleString() }}
                 
                 </span>
              </a>
            </div>
          
          </div>
    
    </div>

    <div class="flex flex-col flex-grow">
      <!-- MAIN CONTENT ------------------------------ -->

      <div class="flex flex-col flex-grow overflow-au ">

        <editor-content :editor="editor" />
        
      </div>

      <div class="h-16 bg-gray-100 border-t border-gray-300 text-right">
      
        <button
        @click ="saveNote"
         class="bg-grey-500 hover:bg-blue-700 hover:text-white font-bold py-2 px-4 rounded m-3">
            Save Note
        </button>
        
      
      </div>
    
    </div>
  
  </div>
 
</template>

<script>


import { Editor, EditorContent } from '@tiptap/vue-3'
import StarterKit from '@tiptap/starter-kit'

export default {
  components: {
    EditorContent,
  },

  data() {
    return {
      editor: null,
      database: null,
      notes: []
    }
  },

  async created(){
    // window.indexedDB.open("notes");
    this.database = await this.getDatabase();
    let notes = await this.getNotes();
    this.notes = notes.reverse();
  },

  mounted() {
    this.editor = new Editor({
      content: '',
      extensions: [
        StarterKit,
      ],
      editorProps: {
        attributes:{
          class: "prose my-6 mx-auto focus:outline-none bg-blue-700 text-white "
        }
      }
    })
  },

  beforeUnmount() {
    this.editor.destroy()
  },

  methods: {
    async getDatabase()
    {
      return new Promise((resolve,reject)=> 
      {
        let db = window.indexedDB.open("notes", 2);

        db.onerror = function() {
          reject('Error openind DB')
        };

        db.onsuccess = e => {
          console.log('db.onsuccess',e)
          resolve(e.target.result);
        };

        db.onupgradeneeded = e => {
          console.log('db.onupgradeneeded',e);

          e.target.result.deleteObjectStore("notes");
          e.target.result.createObjectStore("notes", {keyPath: "created"});
        };
      });
    },

    async saveNote() {
      return new Promise((resolve)=>{
          let transaction = this.database.transaction("notes", 'readwrite');
         
          transaction.oncomplete = () => {
            resolve();
          }

          let now = new Date();
          let note = {
            content: this.editor.getHTML(),
            created: now.getTime()
          };

          this.notes.unshift(note);


          transaction.objectStore("notes").add(note);
          console.log(note)
      });
      
    },
  


  async getNotes(){
    return new Promise((resolve)=> {
      // let transaction = this.database.transaction('notes');
      // let notesStore = transaction.objectStore('notes');
      // let notes = notesStore.getAll();
      //   notes.onsuccess = e => {
      //   resolve(e.target.result);
      // }



      this.database.transaction('notes')
            .objectStore('notes')
            .getAll()
            .onsuccess = e => {
              console.log('getNotes()', e.target.result);
              resolve(e.target.result)
              };
    });
  }
}
}






</script>

<style>

</style>
