<template>
  <div id="app">
    <div>
      TODO: edit items after creating
    </div>
    <div class="add-record-form">
      <label class="form-label">Title:</label>
      <input class="form-stretch" v-model="currentRecord.title">

      <label class="form-label">Category:</label>
      <select class="form-stretch" v-model="currentRecord.category.id">
        <option disabled value=-1>Выберите один из вариантов</option>
        <option v-for="option in masterData.categoryStore" :key="option.id" :value="option.id">{{ option.title }}</option>
      </select>
    
      <label class="form-label">Tags:</label>
      <select class="form-stretch" @change="addTag" v-model="currentRecord.currentTag">
        <option disabled value=null>Выберите один из вариантов</option>
        <option v-for="option in masterData.tagStore" :key="option" :value="option">{{ option }}</option>
      </select>
      <div><span v-for="tag in currentRecord.tags" :key="tag" class="tag">{{ tag }}<button @click="deleteTag(tag)">x</button></span></div>
      
      <button type="button" class="form-right" @click="addItem" :disabled="!canAddItem">Add</button>
    </div>
    <div class="item-list-form">
      <div class="item-list-search">
        <label for="search">Search:</label>
        <input name="search" type="text" v-model="search">
      </div>
      <div class="item-list-filter">
        <span class="item-list-filter-tag" v-for="option in masterData.tagStore" :key="option" >
          <input type="checkbox" :id="option" :value="option" v-model="filters">
          <label :for="option">{{ option }}</label>
        </span>
      </div>
      <div class="item-list-form" v-for="category in itemsByCategory" :key="category.category" >
        <div class="category-name">{{ getCategoryName(category) }} ({{ category.list.length }})</div>
        <div class="item-list" v-for="item in category.list" :key="item.id">
          <div>
            <input type="checkbox" v-model="item.done">
            <span class="item-list-title">{{ item.title }}</span>
            <button @click="deleteItem(item)">x</button>
          </div>
          <div><span v-for="tag in item.tags" :key="tag" class="tag">{{ tag }}</span></div>
        </div>
      </div>
    </div>
    <div class="source-data-form">
      <div class="form-right">
        <button type="button" @click="save">Save</button>
        <button type="button" @click="restore">Restore</button>
      </div>
      <textarea class="form-stretch" rows="5" v-model="savedData"></textarea>
    </div>
  </div>
</template>

<script>

export default {
  name: 'App',
  data() {
    return {
      store: [],
      filters: [],
      search: '',
      savedData: '',
      currentRecord: {
        title: '',
        category: { id: -1, title: '' },
        currentTag: null,
        tags: []
      },
      masterData: {
        categoryStore: [{
          id: 1,
          title: 'Бытовые'
        }, {
          id: 2,
          title: 'Простые'
        }, {
          id: 3,
          title: 'Не простые'
        }, {
          id: 4,
          title: 'Труднодостижимые'
        }, {
          id: 5,
          title: 'Нереальные'
        }],
        tagStore: [
          'Удовольствия, веселье, досуг', 
          'Интересная, активная жизнь', 
          'Здоровье', 
          'Семья', 
          'Дружба', 
          'Любовь', 
          'Внутренняя гармония', 
          'Комфортная безбедная жизнь',
          'Самоуважение',
          'Чувство достижения',
          'Уважение общества',
          'Безопасность',
          'Мудрость',
          'Свобода',
          'Справедливость',
          'Вечная жизнь, спасение',
          'Мир во всем мире',
          'Понимание красоты'
        ]
      }
    }
  },
  computed: {
    canAddItem() {
      return this.currentRecord.title && this.currentRecord.category.id >= 0 && this.currentRecord.tags.length > 0
    },
    itemsFiltered() {
      let result = this.store
      if (this.search) {
        result = result.filter(x => x.title.includes(this.search))
      }
      if (this.filters.length) {
        return result.filter(x => this.filters.some(y => x.tags.includes(y)))
      } else {
        return result
      }
    },
    itemsByCategory() {
      let dict = {}
      this.itemsFiltered.forEach(el => {
        dict[el.category] = dict[el.category] || []
        dict[el.category].push(el)
      })
      let result = []
      for (let k of Object.keys(dict)) {
        let i = { category: k, list: dict[k] }
        result.push(i)
      }
      return result
    }
  },
  methods: {
    addItem() {
      let item = {
        done: false,
        title: this.currentRecord.title,
        category: this.currentRecord.category.id,
        tags: this.currentRecord.tags
      }
      this.store.push(item)
      this.resetCurrentRecord()
      this.save()
    },
    addTag() {
      if (!this.currentRecord.tags.includes(this.currentRecord.currentTag)) {
        this.currentRecord.tags.push(this.currentRecord.currentTag)
        this.currentRecord.currentTag = null
      }
    },
    getCategoryName(item) {
      return this.masterData.categoryStore.find(x => x.id == item.category).title
    },
    deleteItem(item) {
      let index = this.store.indexOf(item)
      if (index >= 0) {
        this.store.splice(index, 1)
        this.save()
      }
    },
    deleteTag(tag) {
      let index = this.currentRecord.tags.indexOf(tag)
      if (index >= 0) {
        this.currentRecord.tags.splice(index, 1)
      }
    },
    resetCurrentRecord() {
      this.currentRecord.title = ''
      this.currentRecord.category = { id: -1, title: '' }
      this.currentRecord.tags = []
    },
    save() {
      let data = JSON.stringify(this.store)
      this.savedData = data
      localStorage['todo-item-data'] = data
    },
    restore() {
      if (this.savedData) {
        let data = JSON.parse(this.savedData)
        this.store = data
      } else {
        console.warn('no data to restore')
      }
    }
  },
  mounted() {
    let dataStr = localStorage['todo-item-data'] || ''
    if (dataStr) {
      let data = JSON.parse(dataStr)
      this.store = data
    } else {
      console.warn('no data to restore')
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.add-record-form {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  width: 900px;
  margin-left: auto;
  margin-right: auto;
}
.item-list-form {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  width: 900px;
  margin-left: auto;
  margin-right: auto;
  margin-top: 20px;
  margin-bottom: 20px;
}
.source-data-form {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  width: 900px;
  margin-left: auto;
  margin-right: auto;
}
.form-label {
  margin-top: 10px;
}
.form-stretch {
  align-self: stretch;
}
.form-right {
  align-self: flex-end;
  margin: 10px;
}
.category-name {
  font-size: 20px;
}
.tag {
  display: inline-block;
  border-style: dashed;
  border-width: thin;
  border-color: lightgray;
  margin: 3px;
  padding: 3px;
  font-size: 12px;
}
.tag button {
  margin-left: 2px;
  padding: 2px;
}
.item-list-title {
  margin-left: 10px;
  margin-right: 10px;
  font-weight: bold;
}
.item-list {
  padding: 15px;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}
.item-list button {
  padding: 2px;
}
.item-list-filter-tag {
  display: inline-block;
  border-width: thin;
  border-style: ridge;
  padding: 3px;
  margin: 3px;
  font-size: 12px;
}
.item-list-filter-tag input {
  height: 10px;
}
.item-list-search {
  margin-top: 10px;
  margin-bottom: 10px;
}
.item-list-search input {
  margin-left: 10px;
}
</style>
