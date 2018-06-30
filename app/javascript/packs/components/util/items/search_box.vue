<template>
  <div id='root'>
    <form>
      <input type='text' :placeholder="`Procurar ${item_type}`" v-model='search_string' />
    </form>
    <div class='content'>
      <draggable v-model="search_results" :options="draggable_options" :move='drag_check' v-if='!show_info'>
        <div v-for='(result, index) in search_results' :key='index'>
          <div class='search-result'>
            <move-icon title='Mover' rootClass='handle' w='20' h='20' />
            <div class='info'>
              <div class='main'>{{result.code}}</div>
              <div class='sub'>{{result.name}}</div>
            </div>
            <div class='spacer'/>
            <div v-on:click="request_subject_info(result.id)" class='button-container'>
              <div class='spacer'/>
              <information-circle-icon title='Informações' rootClass='header-icon' w='25' h='25' />
              <div class='spacer'/>
            </div>
          </div>
        </div>
      </draggable>
      <div class='subject-info' v-if='show_info'>
        <div class='header'>
          <div v-on:click="show_info = false" class='close-button'>
            <close-icon title='Fechar' rootClass='header-icon' w='30' h='30' />
          </div>
          <div class='spacer'/>
        </div>
        <div class='header'>
          <button v-on:click='set_progress(0)'>Não feito</button>
          <button v-on:click='set_progress(1)'>Fazendo</button>
          <button v-on:click='set_progress(2)'>Feito</button>
        </div>
        <h3 class='title'>{{subject.code}}</h3>
        <h5 class='title'>{{subject.name}}</h5>
        <div class='title'>{{subject.credits_class}} créditos aula</div>
        <div class='title'>{{subject.credits_work}} créditos trabalho</div>
        <div class='desc'>{{subject.description}}</div>
      </div>
    </div>
  </div>
</template>

<script>
import auth from '../auth.js';

import draggable from 'vuedraggable';
import MoveIcon from 'vue-ionicons/dist/md-move.vue';
import CloseIcon from 'vue-ionicons/dist/md-close.vue';
import InformationCircleIcon from 'vue-ionicons/dist/md-information-circle.vue';

export default {
  name: 'search-box',
  props: {
    item_type: { default: '' }
  },
  components: {
    draggable,
    MoveIcon,
    CloseIcon,
    InformationCircleIcon
  },
  mounted() {
    this.search_string = '';
    window.bus.$on('request-subject-info', this.subject_info);
  },
  data() {
    return {
      subject: { code: 'AAA0000', name: 'Lorem ipsum' },
      show_info: false,
      search_string: null,
      search_results: [],
      // Options for Vue.Draggable
      draggable_options: {
        animation: 200,
        handle: '.handle',
        group: {
          name: 'item',
          pull: 'clone',
          put: false
        },
        sort: false
      }
    };
  },
  watch: {
    search_string() {
      this.show_info = false;
      auth
        .request('subject/list', { search: this.search_string })
        .then(result => {
          console.log(result.data);
          this.search_results = result.data.subjects;
        });
    }
  },
  methods: {
    set_progress(progress) {
      auth
        .request('subject/update', { subject_id: this.subject.id, progress })
        .then(() => {
          window.bus.$emit('reload-groups');
        });
    },
    subject_info(data) {
      let subject_id = data.subject_id;
      if (subject_id == null) this.show_info = false;
      auth.request('subject/fetch', { subject_id }).then(result => {
        this.subject = result.data.subject;
        this.show_info = true;
      });
    },
    request_subject_info(subject_id) {
      window.bus.$emit('request-subject-info', { subject_id });
    },
    // Checks if can drag
    drag_check(evn, origEvt) {
      return (
        evn.relatedContext.element && evn.relatedContext.element.can_modify
      );
    }
  }
};
</script>

<style lang="scss" scoped>
/********************
 * Formulary inputs *
 ********************/

$input-field-padding: 5pt;
$input-field-margin: 10pt;
$input-field-border-radius: 8pt;
// Text input styling
form {
  display: flex;
}

form input {
  flex: 1;
  padding: $input-field-padding;
  margin: $input-field-margin;
  border-radius: $input-field-border-radius;
  border: none;
}

// Select input styling
form select {
  padding: $input-field-padding;
  margin: $input-field-margin;
  border-radius: $input-field-border-radius;
  border: none;
}

form select .select-items {
  border-top-left-radius: 5pt;
  border-top-right-radius: 5pt;
}

// Remove arrows from number input
input[type='number']::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

/******************
 * Search Results *
 ******************/
.search-result {
  display: flex;
  padding: 8pt;
  vertical-align: middle;
  border-top: solid #ddd 1px;
  border-bottom: solid #ddd 1px;
}

/****************
 * Subject info *
 ****************/
.header {
  display: flex;
  border-bottom: solid 1px #ddd;
}

.header > button {
  flex: 1;
  background-color: #333;
  color: #ddd;
  border: solid #ddd 1px;
  cursor: pointer;
}
.header > button:hover:active {
  background-color: #444;
}

.header > button:hover {
  background-color: #222;
}

.handle:hover {
  color: white;
}

.close-button:hover {
  color: white;
}

.title {
  text-align: center;
}

.desc {
  text-align: justify;
  padding: 15pt;
}

/********
 * Misc *
 ********/

.content {
  display: flex;
  flex-direction: column;
  width: 100%;
  flex-grow: 1;
  color: #ddd;
}

.handle {
  margin: auto;
  cursor: pointer;
  flex-shrink: 1;
}

.button-container {
  display: flex;
  flex-direction: column;
}

.header-icon {
  margin: auto;
  cursor: pointer;
  flex-shrink: 1;
}

.header-icon:hover {
  color: white;
}

.info {
  padding: 5pt;
}

.info .sub {
  font-size: 9pt;
}
</style>