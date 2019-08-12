<template>
  <div class="container">
    <h1>{{ title }}</h1>
    <h2>Total notes:{{ counter }}</h2>

    <div class="form">
      <div class="form-group">
        <label>Note title</label>

        <input
          class="form-control"
          type="text"
          v-model="note.title"
        >
        <span class="text-danger" v-if="validationErrors.title" v-text="validationErrors.title"></span>
      </div>
      <div class="form-group">
        <label>Note text</label>
        <input
          class="form-control"
          type="text"
          v-model="note.text"
        >
        <span class="text-danger" v-if="validationErrors.text" v-text="validationErrors.text"></span>
      </div>
      <div class="form-group">
        <label>Note email</label>
        <input
          class="form-control"
          type="text"
          v-model="note.email"
        >
        <span class="text-danger" v-if="validationErrors.email" v-text="validationErrors.email"></span>
      </div>
      <button
        class="btn btn-primary"
        :disabled=isDisabled
        @click="addNote">Submit
      </button>
    </div>

    <div class="myBtnContainer">
      <button :class="sortParam==='date' ? 'btn btn-outline-secondary active' : 'btn btn-outline-secondary '" @click="sortParam='date'">From date</button>
      <button :class="sortParam==='title' ? 'btn btn-outline-secondary active' : 'btn btn-outline-secondary '" @click="sortParam='title'">From title</button>
    </div>

      <div class="col-sm-4 note" v-for="(note, index) in sortedList">
        <div class="card">
          <div class="card-block">
            <button class="close" @click="removeNote(index)">x</button>
            <h4 class="card-title">{{ note.title }}</h4>
            <h6 class="card-subtitle mb-2 text-muted">{{note.date}}</h6>
            <p class="card-text">{{note.email}}</p>
            <p class="card-text">{{note.text}}</p>
            <button class="btn btn-primary" @click="copy(index)">Дублировать</button>
          </div>
        </div>
      </div>
    </div>

</template>

<script>
export default {
  name: 'Notes',
  data () {
    return {
      title: 'Notes',
      note: {
        title: '',
        text: '',
        email: ''
      },
      notes: [],
      counter: 0,
      validationErrors: {
        text: null,
        title: null,
        email: null
      },
      isDisabled: true,
      dateNow: new Date(),
      sortParam: 'date',
      desc: true
    }
  },
  created () {
    let formatted = this.dateToDMDY(this.dateNow)
    this.notes = [{
      title: 'Lunch at 12PM',
      text: 'Call Robert to remember him.',
      email: 'test@gmail.com',
      date: formatted
    }, {
      title: 'Gym with Jane',
      text: 'She is needing your motivation...',
      date: formatted,
      email: 'test@gmail.com'
    }, {
      title: 'Call mom',
      text: 'Ask her about the cats?',
      date: formatted,
      email: 'test@gmail.com'
    }]

    this.counter = this.notes.length
  },
  computed: {

    sortedList () {
      switch (this.sortParam) {
        case 'title':
          this.notes.sort(function (d1, d2) {
            return (d1.title.toLowerCase() > d2.title.toLowerCase()) ? 1 : -1
          })
          break
        case 'date':
          this.notes.sort(function (d1, d2) {
            return new Date(d1.date) ? -1 : new Date(d2.date) ? 1 : 0
          })
          break
      }
      return this.notes
    }
  },
  watch: {
    note: {
      handler: function (item) {
        let char = item.title.slice(0, 1)
        if (item.title.length <= 3 || char === char.toLowerCase()) {
          this.validationErrors.title =
                            'Title must have contain more than three characters and start with a capital letter'
        } else this.validationErrors.title = null
        if (item.text.length <= 3) this.validationErrors.text = 'Text must have contain more than three characters'
        else this.validationErrors.text = null

        let emailReg = /\S+@\S+\.\S+/
        if (!emailReg.test(item.email)) this.validationErrors.email = 'Email is fail'
        else this.validationErrors.email = null

        if (!this.validationErrors.title && !this.validationErrors.text && !this.validationErrors.email) {
          this.isDisabled = false
        }

        if (item.title.length === 0 && item.text.length === 0 && item.email.length === 0) {
          this.validationErrors.title = null
          this.validationErrors.text = null
          this.validationErrors.email = null
        }
      },
      deep: true
    }
  },
  methods: {
    addNote () {
      let { text, title, email } = this.note
      this.notes.push({
        text,
        title,
        date: this.dateToDMDY(this.dateNow),
        email
      })
      this.counter = this.notes.length
      this.restForm()
    },
    removeNote (index) {
      this.notes.splice(index, 1)
    },
    restForm () {
      this.note = {
        title: '',
        text: '',
        email: ''
      }
    },
    dateToDMDY (date) {
      let d = date.getDate()
      let m = date.getMonth() + 1
      let y = date.getFullYear()
      return (d <= 9 ? '0' + d : d) + '.' + (m <= 9 ? '0' + m : m) + '.' + y
    },
    copy (index) {
      let note = this.notes.filter(function (elem, i) {
        return i === index
      })
      let newNote = {}
      newNote.title = note[0].title + ' (copy)'
      newNote.email = note[0].email
      newNote.text = note[0].text
      newNote.date = this.dateToDMDY(this.dateNow)

      this.notes.push(newNote)
    }

  }
}
</script>

<style scoped lang="scss">
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin: 20px;
  }

  .form {
    text-align: left;
  }

  .card {
    text-align: left;
    border: 1px solid #2c3e50;
    border-radius: 4px;
    padding-left: 8px;
    padding-right: 8px;
    padding-bottom: 10px;
  }
  .myBtnContainer {
    button {
      margin-right: 10px;
    }
  }

  .note {
    padding: 5px;
  }
</style>
