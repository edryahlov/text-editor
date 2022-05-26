<script setup>
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://vuejs.org/api/sfc-script-setup.html#script-setup
import HelloWorld from './components/HelloWorld.vue'

import Quill from "quill";
import "quill/dist/quill.core.css";
import "quill/dist/quill.bubble.css";
import "quill/dist/quill.snow.css";
import {ref, onMounted, onUnmounted, onUpdated, watch} from "vue";

const options = JSON.parse(JSON.stringify([
  {
    "id": "tags",
    "name": "Добавить метку - \"мм\" или \"tt\"",
    "links": ["tags"]
  },
  {
    "id": "links",
    "name": "Добавить связь - \"сс\" или \"ll\"",
    "links": ["links"]
  },
]));
const tags = JSON.parse(JSON.stringify([
  {
    "id": "tag-id-1",
    "name": "Метка 1",
    "created": 1622699295261,
    "updated": 1622699295261,
    "links": ["ib-id-1", "ib-id-2"]
  },
  {
    "id": "tag-id-2",
    "name": "Метка 2",
    "created": 1622699295261,
    "updated": 1622699295261,
    "links": ["ib-id-2"]
  },
  {
    "id": "tag-id-3",
    "name": "Метка 3",
    "created": 1622699295262,
    "updated": 1622699295262,
    "links": ["ib-id-3"]
  }
]));
const links = JSON.parse(JSON.stringify([
  {
    "id": "ib-id-1",
    "title": "Link1",
    "content": "...",
    "created": 1624019097656,
    "updated": 1624019476395,
    "links": [],
    "tags": [
      {"name":"Метка 1","id":"tag-id-1"},
      {"name":"Метка 2","id":"tag-id-2"}
    ]
  },
  {
    "id": "ib-id-2",
    "title": "Link2",
    "content": "...",
    "created": 1624019097656,
    "updated": 1624019476395,
    "links": ["ib-id-1"],
    "tags": [
      {"name":"Метка 2","id":"tag-id-2"}
    ]
  }
]));
const titles = {
  'options': 'Введи или выбери триггер и нажми "TAB"',
  'tags': 'Выбрать - "TAB", создать - "ENTER"',
  'links': 'Выбрать - "TAB", создать - "ENTER"',
}

let popupSections = {options, tags, links};
const popupSectionToShow = ref('options');
const popupActiveOption = ref({'options': 0, 'tags': 0, 'links': 0});

const popupVisible = ref(false);
const popupX = ref(0);
const popupY = ref(0);
let input = ref('');

let quill;

document.addEventListener('keydown', (e) => {
  if (e.code === "ArrowUp" && popupVisible.value) {
    e.preventDefault();
    if (popupActiveOption.value[popupSectionToShow.value] === 0) {
      popupActiveOption.value[popupSectionToShow.value] = popupSections[popupSectionToShow.value].length-1;
    } else {
      popupActiveOption.value[popupSectionToShow.value] -= 1;
    }
  }
  else if (e.code === "ArrowDown" && popupVisible.value) {
    e.preventDefault();
    if (popupActiveOption.value[popupSectionToShow.value] >= popupSections[popupSectionToShow.value].length-1) {
      popupActiveOption.value[popupSectionToShow.value] = 0;
    } else {
      popupActiveOption.value[popupSectionToShow.value] += 1;
    }
  }
  else if (e.code === "Escape" && popupVisible.value) {
    e.preventDefault();
    popupVisible.value = false;
  }
  else if (e.code === "Enter" && popupVisible.value) {
    e.preventDefault();
    console.log('enter2', e.code === "Enter", popupVisible.value)
    if (input.value.length > 0) {
      triggerDialog(input.value);
    }
  }
  else if (e.code === "Tab" && popupVisible.value) {
    e.preventDefault();

    // quill.focus();
    const selected = popupSections[popupSectionToShow.value]
        .find((el, index) => index === popupActiveOption.value[popupSectionToShow.value]);

    if (popupSectionToShow.value === 'options') {
      selectOption(selected.id);
    } else {
      const key = Object.keys(selected).includes('name') ? 'name' : 'title';
      triggerDialog(selected[key]);
    }
  }
  else if (popupVisible.value && (popupSectionToShow.value === 'tags' || popupSectionToShow.value === 'links')) {
    if (e.key.length === 1) {
      input.value += e.key;
    } else if (e.key === 'Backspace') {
      input.value = input.value.substr(0, input.value.length - 1)
    }
    const pattern = new RegExp(input.value, "gi");
    popupSections['tags'] = tags.filter(el => pattern.test(el.name));
    popupSections['links'] = links.filter(el => pattern.test(el.title));
  }

})

watch(() => popupVisible.value, visible => {
  if (visible) {
    // quill.enable(!visible);
  } else {
    quill.enable(true);
    quill.focus();
    popupSectionToShow.value = 'options'
  }
})

function triggerDialog(text = '') {
  popupVisible.value = !popupVisible.value;

  if (!popupVisible.value) {
    popupSections['tags'] = tags;
    popupSections['links'] = links;
  }
  // const caretPosition = quill.getSelection() || {index: 0, length: 0};

  if (popupSectionToShow.value === 'tags') {
    const shift = input.value.length > 0 ? 2 : 1;
    quill.deleteText(quill.getSelection().index - (input.value.length + shift), input.value.length + shift);
    quill.insertText(quill.getSelection(), `#${text}`);
  }
  else if (popupSectionToShow.value === 'links')  {
    quill.deleteText(quill.getSelection().index -= 1, 1);
    quill.insertText(quill.getSelection(), text, 'link', 'https://world.com');
  }
  const caretPositionInPixels = quill.getBounds(quill.getSelection());
  popupX.value = caretPositionInPixels.left;
  popupY.value = caretPositionInPixels.top;
  input.value = '';
}

function selectOption(option) {
  if (option === 'tags' || option === 'links') {
    popupSectionToShow.value = option;
  } else {
    const selected = popupSections[popupSectionToShow.value].find(el => el.id === option);
    const key = Object.keys(selected).includes('name') ? 'name' : 'title';
    triggerDialog(selected[key]);
    popupSectionToShow.value = 'options';
  }
}

onMounted(() => {
  quill = new Quill('#editor', {
    modules: {
      keyboard: {
        bindings: {
          'slash': {
            key: '/',
            handler: function(e) {
              const caretPosition = quill.getSelection() || {index: 0, length: 0};
              quill.insertText(caretPosition, '/');
              triggerDialog();
            }
          },
          'tab': {
            key: 9,
            handler: function(range, event) {
              // moveSelected(event.event.code);
            }
          },
          'enter': {
            key: 13,
            handler: function(range, event) {
              console.log('enter')
              // moveSelected(event.event.code);
            }
          },
        }
      },
    },
    theme: "snow",
    placeholder: "Type something in here!",
  });
})

onUnmounted(() => {
  document.removeEventListener('keydown', () => console.log('removed'))
})

onUpdated(() => {
  document.removeEventListener('keydown', () => console.log('removed'))
})

</script>

<template>
  <div class="editor-wrap">

    <div class="editor">
      <div id="editor"></div>
    </div>

<!--    {{popupVisible}}<br>-->
<!--    {{popupSectionToShow}}<br>-->
    {{input}}<br><br>
    {{popupSections[popupSectionToShow]}}

    <q-card ref="card" class="my-card" v-if="popupVisible" :style="{'left': (popupX+5)+'px', 'top': (popupY+60)+'px'}">
        <q-card-section>
          <div class="text-subtitle2">{{titles[popupSectionToShow]}}</div>
        </q-card-section>

        <q-card-actions vertical>
          <template v-for="(item, index) in popupSections[popupSectionToShow]" :key="item.id">
            <q-btn
                flat
                :label="item.name || item.title"
                :class="{'active': popupActiveOption[popupSectionToShow] === index}"
                @click="selectOption(item.id)"
            />
            <q-separator v-if="index !== popupSections[popupSectionToShow].length-1" />
          </template>
        </q-card-actions>
    </q-card>
  </div>

</template>

<style lang="sass" scoped>
.editor
  width: 60%
  &-wrap
    margin: 30px
    position: relative
.my-card
  position: absolute
  display: inline-block
.active
  background: #dedede
</style>
