<template>
    <div class="block-content">
        <h3 class="title-list">
            Список заметок
        </h3>
        <ul class="block-notesblock-notes">
            <li class="block-note" 
               v-for="data in arrDataSto" 
               :key="data.title" 
               :style="{background : data.color}">
                <ul class="note-categorys">
                    <li class="note-category" 
                       v-for="category in data.category" 
                       :style="{background : category.color}">
                        {{ category.category }}
                    </li>
                </ul>
                <div class="block-icons" 
                   :data-key="data.key">
                    <div class="save-note d-none" 
                    title="Сохранить заметку" 
                    @click="saveNote($event)">
                        &#10004;
                    </div>
                    <div class="edit-note" 
                    title="Редактировать заметку" 
                    @click="updateNote($event)">
                        &#9998;
                    </div>
                    <div class="delete-note" 
                    title="Удалить заметку" 
                    @click="deleteNote($event)">
                        &#10006;
                    </div>
                </div>
                <h4 class="note-title">
                    {{data.title}}
                </h4>
                <div class="note-text">
                    {{data.text}}
                </div>
                <ul class="note-labels d-flex flex-wrap-wrap">
                    <li class="label-note" 
                       v-for="label in data.label" 
                       :style="{background : label.color}" 
                       :key="label.text">
                        {{ label.text }}
                    </li>
                </ul>
            </li>
        </ul>
    </div>
</template>

<script>
    import {
        eventHub
    } from '../main'
    export default {
        name: 'ListNotes',
        data: function() {
            return {
                key: null,
                dataObj: {},
                dataSave: {},
                myStorage: null,
                arrDataSto: [],
                parentElem: null
            }
        },
        methods: {
            getLocal: function() {
                
                this.arrDataSto = []
                this.myStorage = window.localStorage
                
                for (let key in this.myStorage) {
                    if (key.split(';')[0] == 'data') {
                        this.dataObj = JSON.parse(localStorage.getItem(key))
                        this.dataObj.key = key
                        this.arrDataSto.push(this.dataObj)
                    }
                }
                
            },
            deleteNote: function(e) {
                
                localStorage.removeItem(e.target.parentNode.getAttribute('data-key'))
                this.getLocal()
                
            },
            updateNote: function(e) {
                
                e.target.classList.add('d-none')
                this.parentElem = e.target.parentNode.parentNode;
                this.parentElem.childNodes[1].childNodes[0].classList.remove('d-none')
                this.parentElem.childNodes[2].classList.add('blue-border')
                this.parentElem.childNodes[2].setAttribute('contenteditable', 'true')
                this.parentElem.childNodes[3].classList.add('blue-border')
                this.parentElem.childNodes[3].setAttribute('contenteditable', 'true')
                
            },
            saveNote: function(e) {
                
                e.target.classList.add('d-none')
                this.parentElem = e.target.parentNode.parentNode;
                this.parentElem.childNodes[1].childNodes[1].classList.remove('d-none')
                this.parentElem.childNodes[2].classList.remove('blue-border')
                this.parentElem.childNodes[2].setAttribute('contenteditable', 'false')
                this.parentElem.childNodes[3].classList.remove('blue-border')
                this.parentElem.childNodes[3].setAttribute('contenteditable', 'false')
                this.sendDataLocalStr(this.parentElem)
                
            },
            sendDataLocalStr: function(parentElem) {
                
                this.key = parentElem.childNodes[1].getAttribute('data-key')
                this.dataSave = JSON.parse(localStorage.getItem(this.key))
                this.dataSave.title = parentElem.childNodes[2].innerText
                this.dataSave.text = parentElem.childNodes[3].innerText
                localStorage.removeItem(this.key)
                localStorage.setItem(this.key, JSON.stringify(this.dataSave))
                
            }
        },
        mounted() {
            
            this.getLocal()
            eventHub.$on('updatedata', this.getLocal)
            
        },
    }
</script>