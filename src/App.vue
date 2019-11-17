<template>
    <div id="app">
        <div class="container">
            <div class="row">
                <h1 class="title-list">
                    Создать заметку
                </h1>
            </div>
            <div class="row">
                <div class="block-add-note">
                    <div class="block-add-note-head">
                        <label for="add-title" 
                        class="title">
                            Добавить заглавие заметки
                        </label>
                        <input type="text" 
                        id="add-title" 
                        ref="addtitle">
                        <label for="add-text" class="title">
                            Добавить текст заметки
                        </label>
                        <textarea 
                        id="add-text" 
                        name="add-text" 
                        ref="addtext" 
                        @input="removeClassErr($event)"></textarea>
                        <div class="d-flex justify-content-between">
                            <span class="title text-left">
                                Задать цвет заметки
                            </span>
                            <ul class="block-colors">
                                <li v-for="(color, index) in arrColors" 
                                :key="index" 
                                :class="{'active-color': activeColor === index}" 
                                :data-index="index" 
                                :data-color="color" 
                                :style="{background : color}" 
                                @click="getColor(1, $event)" 
                                class="add-color"></li>
                            </ul>
                        </div>
                        <h5 class="title">
                            Добавить категорию к заметке
                        </h5>
                        <ul class="list-categorys d-flex justify-content-center" ref="categorys">
                            <li v-for="category in varCategorys" 
                               class="category" 
                               :key="category.text" 
                               :data-color="category.color" 
                               :data-categorie="category.text" 
                               :style="{background : category.color}" @click="categorysToggle($event)">
                                   {{ category.text }}
                            </li>
                        </ul>
                        <h5 class="title">
                            Добавить ярлык к заметке
                        </h5>
                        <div class="d-flex">
                            <span class="title text-left">
                                Выбрать цвет ярлыка
                            </span>
                            <ul class="block-colors">
                                <li v-for="(color, index) in arrColorsNote" 
                                :key="color" 
                                :data-index="index" 
                                @click="getColor(2, $event)" 
                                :class="{'active-color': activeLabel === index}" 
                                :data-color="color" 
                                :style="{background : color}" 
                                class="add-color"></li>
                            </ul>
                        </div>
                        <h4 for="add-text-label" class="title">
                            Добавить текст к ярлыку
                        </h4>
                        <div class="d-flex">
                            <input type="text" 
                            id="add-text-label" 
                            ref="textlabel">
                            <button id="btn-add-label" 
                            @click="addLabel()">
                                Добавить ярлык
                            </button>
                        </div>
                        <ul ref="labels" class="d-flex block-labels">
                            <li v-for="(item, index) in arrLabels" 
                            :style="{background : item.color}" 
                            class="label-note" 
                            @click="toggleLebal($event)" 
                            :key="index">
                                {{ item.text }}
                            </li>
                        </ul>
                        <div class="block-btn">
                            <button id="btn-add-note" 
                            @click="createObjLocalStorage()">
                                Сохранить заметку
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div id="notes">
            <ListNotes @updatedata="getLocal()"></ListNotes>
        </div>
    </div>
</template>

<script>
    import ListNotes from './components/ListNotes.vue'
    import {
        eventHub
    } from './main'

    export default {
        name: 'app',
        data: function() {
            return {
                key: null,
                colSavLabel: null,
                active: null,
                date: null,
                childs: [],
                sendData: false,
                activeColor: 0,
                activeLabel: 0,
                categoryObj: {},
                labels: [],
                arrCategory: [],
                colorLabel: 'rgba(255, 1, 200, 0.6)',
                colorNote: 'aliceblue',
                labelSetObj: {},
                arrColors: ['aliceblue', 'rgba(231, 250, 4, 0.7)', 'rgba(0, 186, 255, 0.7)', 'rgba(6, 255, 63, 0.7)', 'rgba(247, 127, 7, 0.7)'],
                arrColorsNote: ['violet', 'pink', 'aqua', 'aquamarine', 'chartreuse'],
                varCategorys: [{
                        text: 'друзья',
                        color: 'rgba(255, 255, 0, 0.5)'
                    },
                    {
                        text: 'работа',
                        color: 'rgba(0, 255, 255, 0.5)'
                    },
                    {
                        text: 'отдых',
                        color: 'rgba(29, 0, 255, 0.5)'
                    },
                    {
                        text: 'хобби',
                        color: 'rgba(0, 255, 10, 0.5)'
                    },
                    {
                        text: 'спорт',
                        color: 'rgba(255, 157, 0, 0.5)'
                    }
                ],
                arrLabels: [],
                dataLocalStor: {},
                categories: null,
                arrAddCategorys: new Map()
            }
        },
        methods: {
            getColor: function(num, e) {
                
                this.active = Number(e.target.getAttribute('data-index'))
                
                if (num == 1) {
                    this.activeColor = this.active
                    this.colorNote = e.target.getAttribute('data-color')
                } else if (num == 2) {
                    this.activeLabel = this.active
                    this.colorLabel = e.target.getAttribute('data-color')
                }
                
            },
            categorysToggle: function(e) {
                
                e.target.classList.toggle('active-color')

                this.categoryObj = {}
                this.categoryObj.category = e.target.getAttribute('data-categorie')
                this.categoryObj.color = e.target.getAttribute('data-color')

                this.addRemoveElement(this.categoryObj)
                
            },
            addRemoveElement: function(item) {

                if (this.arrAddCategorys.has(item.category)) this.arrAddCategorys.delete(item)
                else this.arrAddCategorys.set(item.category, item)

            },
            addLabel: function() {
                
                this.labelSetObj = {}
                
                if (this.$refs.textlabel.value != '') {

                    this.labelSetObj.color = this.colorLabel.trim()
                    this.labelSetObj.text = this.$refs.textlabel.value
                    this.$refs.textlabel.value = ''
                    this.arrLabels.push(this.labelSetObj)

                }

            },
            createObjLocalStorage: function() {

                if (this.sendData) {

                    this.getArrLabels()

                    for (let key of this.arrAddCategorys.keys()) {
                        this.arrCategory.push(this.arrAddCategorys.get(key))
                    }

                    this.dataLocalStor = {}
                    this.date = new Date()
                    this.dataLocalStor.title = this.$refs.addtitle.value.trim()
                    this.dataLocalStor.text = this.$refs.addtext.value.trim()
                    this.dataLocalStor.color = this.colorNote
                    this.dataLocalStor.category = this.arrCategory
                    this.dataLocalStor.label = this.labels
                    this.addNoteLocalStorage()

                } else {
                    this.$refs.addtext.classList.add('error-color')
                }
                
            },
            addNoteLocalStorage: function() {

                this.key = `data;${this.date.getMonth()}.${this.date.getDate()}.${this.date.getHours()}:${this.date.getMinutes()}:${this.date.getSeconds()}`
                localStorage.setItem(this.key, JSON.stringify(this.dataLocalStor))
                this.clearData()
                eventHub.$emit('updatedata')
                
            },
            clearData: function() {
                
                this.dataLocalStor = {}
                this.colorNote = 'aliceblue'
                this.$refs.addtitle.value = ' '
                this.$refs.addtext.value = ' '
                this.$refs.textlabel.value = ' '
                this.arrAddCategorys.clear()
                this.arrCategory = []
                this.arrLabels = []
                this.activeLabel = 0
                this.activeColor = 0
                this.sendData = false
                this.childs = this.$refs.categorys.childNodes

                this.childs.forEach(function(elem) {
                    elem.classList.remove('active-color')
                })

            },
            toggleLebal: function(e) {
                
                e.target.classList.toggle('delete-label')
                
            },
            getArrLabels: function() {
                
                this.labels = []
                
                for (let elem of this.$refs.labels.childNodes) {
                    if (!elem.classList.contains('delete-label')) {
                        this.colSavLabel = elem.getAttribute('style').split(':')[1].replace(/\;/g, ' ')
                        this.labels.push({
                            text: elem.innerText,
                            color: this.colSavLabel.trim()
                        })
                    }
                }
                
            },
            removeClassErr: function(e) {
                
                if (e.target.value.trim().length == 0) {
                    this.sendData = false
                    e.target.classList.add('error-color')
                } else {
                    this.sendData = true
                    e.target.classList.remove('error-color')
                }
            }
            
        },
        components: {
            ListNotes
        }
    }
</script>

<style>
    @import './assets/style.css'
</style>