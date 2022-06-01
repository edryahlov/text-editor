<script setup>
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://vuejs.org/api/sfc-script-setup.html#script-setup
// import HelloWorld from './components/HelloWorld.vue'

import Quill from "quill";
import "quill/dist/quill.core.css";
import "quill/dist/quill.bubble.css";
import "quill/dist/quill.snow.css";
import {shallowRef, ref, onMounted, onUnmounted, onUpdated, onBeforeUpdate, watch} from "vue";

const options = JSON.parse(JSON.stringify([
  {
    "id": "tags",
    "name": 'Добавить метку "мм" или "tt"',
    "links": ["tags"]
  },
  {
    "id": "links",
    "name": 'Добавить связь - "сс" или "ll"',
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
  },
  {
    "id": "tag-id-4",
    "name": "Метка 4",
    "created": 1622699295262,
    "updated": 1622699295262,
    "links": ["ib-id-4"]
  },
  {
    "id": "tag-id-5",
    "name": "Метка 5",
    "created": 1622699295262,
    "updated": 1622699295262,
    "links": ["ib-id-5"]
  },
  {
    "id": "tag-id-6",
    "name": "Метка 6",
    "created": 1622699295262,
    "updated": 1622699295262,
    "links": ["ib-id-6"]
  },
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
  'tags': 'Выбрать - "TAB"',
  'links': 'Выбрать - "TAB"',
}

const sourceSections = JSON.parse(JSON.stringify({options, tags, links}));
let popupSections = JSON.parse(JSON.stringify({options, tags, links}));
const popupSectionToShow = ref('options');
const popupActiveOption = ref({'options': 0, 'tags': 0, 'links': 0});

const popupVisible = ref(false);
const popupX = ref(0);
const popupY = ref(0);
let input = ref('');

let quill;

document.addEventListener('keydown', (e) => {
  const card = document.getElementsByClassName('q-card__actions');

  if (e.code === "ArrowUp" && popupVisible.value) {
    e.preventDefault();
    if (popupActiveOption.value[popupSectionToShow.value] === 0) {
      popupActiveOption.value[popupSectionToShow.value] = popupSections[popupSectionToShow.value].length-1;
      if (!!card) card[0].scrollTo(0,1000);
    } else {
      popupActiveOption.value[popupSectionToShow.value] -= 1;
      if (!!card) card[0].scrollTo(0,card[0].scrollTop-40);
    }
  }
  else if (e.code === "ArrowDown" && popupVisible.value) {
    e.preventDefault();
    if (popupActiveOption.value[popupSectionToShow.value] >= popupSections[popupSectionToShow.value].length-1) {
      popupActiveOption.value[popupSectionToShow.value] = 0;
      if (!!card) card[0].scrollTo(0,0);
    } else {
      popupActiveOption.value[popupSectionToShow.value] += 1;
      if (!!card) card[0].scrollTo(0,card[0].scrollTop+40);
    }
  }
  else if (e.code === "Escape" && popupVisible.value) {
    e.preventDefault();
    popupVisible.value = false;
  }
  // else if (e.code === "Enter" && popupVisible.value) {
  //   e.preventDefault();
  //   // quill.disable()
  //
  //   const selected = popupSections[popupSectionToShow.value]
  //       .find((el, index) => index === popupActiveOption.value[popupSectionToShow.value]);
  //
  //   triggerDialog(selected, 'options');
  // }
  else if (e.code === "Tab" && popupVisible.value) {
    e.preventDefault();

    // quill.focus();
    const selected = popupSections[popupSectionToShow.value]
        .find((el, index) => index === popupActiveOption.value[popupSectionToShow.value]);

    triggerDialog(selected, 'options');
  }
  else if (popupVisible.value && e.key.length === 1 && e.key !== '/') {
      input.value += e.key;
  }
  else if (popupVisible.value && e.key === 'Backspace') {
    input.value = input.value.substr(0, input.value.length - 1)
  }
  console.log(popupActiveOption.value)
  // const pattern = new RegExp(input.value, "gi");
  // popupSections[popupSectionToShow.value] = sourceSections[popupSectionToShow.value].filter(el => pattern.test(el.name));
})

function getPopupSectionToShow() {
  const pattern = new RegExp(input.value, "gi");
  return sourceSections[popupSectionToShow.value].filter(el => pattern.test(el.name));
}

// let itemRefs = []
// const setItemRef = el => {
//   if (el) {
//     itemRefs.push(el)
//   }
// }
// onBeforeUpdate(() => {
//   itemRefs = []
// })
// onUpdated(() => {
//   console.log(itemRefs)
// })

watch(() => popupVisible.value, visible => {
  if (visible) {
    // quill.enable(!visible);
  } else {
    quill.enable(true);
    quill.focus();
    popupSectionToShow.value = 'options';
    popupSections[popupSectionToShow.value] = sourceSections[popupSectionToShow.value];
  }
})

function triggerDialog(selected, target = 'tags') {
  let section, key, text;
  section = (!selected) ? 'options' : selected['id'];

  console.log(section, selected, target, popupSectionToShow.value)

  if (section === 'options' && !popupVisible.value) {
    popupVisible.value = true;
    popupSectionToShow.value = section;
  } else if ((section === 'tags' || section === 'links') && popupVisible.value) {
    popupSectionToShow.value = section;
  } else if (target === 'options') {

    key = Object.keys(selected).includes('name') ? 'name' : 'title';
    text = selected[key];

    if (popupSectionToShow.value === 'tags') {
      const shift = input.value.length > 0 ? 2 : 1;
      quill.deleteText(quill.getSelection().index - (input.value.length + shift), input.value.length + shift);
      quill.insertText(quill.getSelection(), `#${text}`);
    }
    else if (popupSectionToShow.value === 'links')  {
      quill.deleteText(quill.getSelection().index -= 1, 1);
      quill.insertText(quill.getSelection(), text, 'link', 'https://world.com');
    }

    popupVisible.value = false;
    popupSectionToShow.value = 'options';
  }

  const caretPositionInPixels = quill.getBounds(quill.getSelection());
  popupX.value = caretPositionInPixels.left;
  popupY.value = caretPositionInPixels.top;
  input.value = '';
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
              // console.log('enter')
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

<!--    {{input}}<br>-->
<!--    {{popupSections[popupSectionToShow]}}<br>-->

    <q-card ref="card" class="my-card" v-if="popupVisible" :style="{'left': (popupX+5)+'px', 'top': (popupY+60)+'px'}">
        <q-card-section>
          <div class="text-subtitle2">{{titles[popupSectionToShow]}}</div>
        </q-card-section>

        <q-card-actions vertical>
          <template
              v-for="(item, index) in getPopupSectionToShow()"
              :key="item.id"
          >
            <q-btn
                flat
                align="left"
                :label="item.name || item.title"
                :class="{'active': popupActiveOption[popupSectionToShow] === index}"
                @click="triggerDialog(item, 'options')"
            />
            <q-separator v-if="index !== popupSections[popupSectionToShow].length-1" style="height: 0;" />
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
  background: #EAEAEA
.q-card
  border-radius: 0
  &__section
    &--vert
      padding: 8px
      font-size: 15px
      color: #959595
  &__actions
    padding: 0
    max-height: 130px
    flex-wrap: nowrap
    overflow-y: auto
.q-btn
  text-transform: none
  font-weight: 400
  font-size: 15px
  min-height: auto
  padding: 8px
  &--rectangle
    border-radius: 0


</style>
