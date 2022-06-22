<template>
  <div class="body">
    <div class="wrapper">
      <div class="description">
        <div class="info">
          <img src="../../../icons/info.png" width="40" alt="">
        </div>
        <span>
          Lorem ipsum dolor sit amet, consectetur adipiscing elit. Mauris dui
          diam, auctor sed placerat id, egestas a purus. Aliquam in orci sit
          amet magna mattis varius</span
        >
      </div>
      <div class="content">
        <div class="text">
          <span
            class="text-items"
            v-for="(item, index) in text"
            :key="`${index}_text`"
          >
            <span class="text-item">{{ item.text }}</span>
            <draggable group="a" :list="item"  :sort="false"  :move="dragHandler" class="abc" style="display: inline-block; padding:10px" >
              <div class="drag-element" :id="item.id">
                <button
                  @click="selectedItem(item.id)"
                  class="no-selected"
                  :class="{ selected: item.buttonValue }"
                  v-if="index < text.length - 1"
                >
                  <span v-if="item.buttonValue">{{ item.buttonValue }}</span>
                  <span v-else style="vertical-align: sub">****</span>
                  <button
                    class="cancel"
                    v-if="item.buttonValue"
                    @click="cancel(item.answerId, item.id)"
                  >
                    <img src="../../../icons/cross.png" alt="" width="20" />
                  </button>
                </button>
              </div>
            </draggable>
          </span>
        </div>
        <div>
          <draggable class="answers" :sort="false" :list="answers" :move="handleDragAnswer" @end="handleDragEnd" group="a">
            <button
              class="answer-btn"
              v-for="(answer, index) in answers"
              :class="{
                'selected-answer': answer.selected,
                disabled: answer.disabled,
              }"
              :key="`${index}_answers`"
              :disabled="answer.disabled"
              @click="selectAnswer(answer.id)"
            >
              {{ answer.label }}
            </button>
          </draggable>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { VueDraggableNext } from "vue-draggable-next";
export default {
  components: {
    draggable: VueDraggableNext,
  },
  data() {
    return {
      content:
        "Liebe Claudia, entschuldige, dass ich mich so lange nicht bei dir gemeldet Wie du weißt, bin ich vor zwei Wochen .... Deswegen hatte ich leider keine Zeit .... meine Freunde. Aber deinen Vorschlag, mal wieder gemeinsam .... Tag miteinander zu verbringen, finde ich sehr gut. Mir passt es auch am besten am Wochenende. Würde es bei dir schon .... Samstag gehen? Da habe ich noch nichts vor. Es würde .... natürlich freuen, wenn du dir bei dieser Gelegenheit auch meine neue Wohnung anschaust. Es ist wunderbar, so viel zu haben. Was meinst du, wir uns bei mir treffen und dann unsere Einkaufstour machen? Oder ist es dir .... , wenn wir zuerst einkaufen und am Abend zusammen kochen und essen? Übrigens, können wir dein Auto nehmen? .... steht schon wieder in der Werkstatt. Schreib mir doch bald, ob du an diesem Tag Zeit hast. .... du keine Zeit hast, finden wir sicher einen anderen Tag.Marion",
      answers: [
        { id: 0, label: "umgezogen", selected: false, disabled: false },
        { id: 1, label: "mich", selected: false, disabled: false },
        { id: 2, label: "einen", selected: false, disabled: false },
        { id: 3, label: "nächsten", selected: false, disabled: false },
        { id: 4, label: "für", selected: false, disabled: false },
        { id: 5, label: "lieber", selected: false, disabled: false },
        { id: 6, label: "wollen", selected: false, disabled: false },
        { id: 7, label: "Meines", selected: false, disabled: false },
      ],
      selectedAnswer: null,
      selectedAnswerId: null,
      text: null,
      currentAnswerIndex: null,
      currentDragId: null,
      debounce: null
    };
  },
  mounted() {
    this.text = this.content.split("....").map((elem, index) => ({
      id: index,
      answerId: null,
      text: elem,
      buttonValue: null,
    }));
  },
  methods: {
    dragHandler() {
      return false;
    },
    selectAnswer(id) {
      this.selectedAnswerId = id
      this.answers.forEach((elem) => {
        if (elem.id === id) {
          this.selectedAnswer = elem.label;
          elem.selected = !elem.selected;
          if (!elem.selected) {
            this.selectedAnswer = null
          }
        } else {
          elem.selected = false;
        }
      });
    },
    selectedItem(id) {
      if (this.selectedAnswer && this.text.find((elem) => elem.id === id).answerId === null) {
        this.answers.find(
          (elem) => elem.label === this.selectedAnswer
        ).disabled = true;
        this.text.forEach((elem) => {
          if (elem.id === id) {
            elem.buttonValue = this.selectedAnswer;
            elem.answerId = this.selectedAnswerId
            this.selectedAnswer = null;
          }
        });
      }
    },
    cancel(answerId, id) {
      this.answers.find((el) => el.id === answerId).disabled = false;
      this.answers.find((el) => el.id === answerId).selected = false;
      this.text.find((elem) => elem.id === id).buttonValue = null;
      this.text.find((elem) => elem.id === id).answerId = null;
    }, 
    handleDragAnswer(e) {
      this.currentAnswerIndex = e.draggedContext.element.id.toString()
      this.currentDragId = e.related.id
      clearTimeout(this.debounce)
      this.debounce = setTimeout(() => {
        this.currentDragId = null
        this.currentAnswerIndex = null
      }, 400)      
        return false;
    },
    handleDragEnd(e) {
      if (this.currentAnswerIndex && this.currentDragId) {
        if(this.text.find((elem) => elem.id === +this.currentDragId).answerId === null) {
          this.text.find((elem) => elem.id === +this.currentDragId).buttonValue = this.answers.find((el) => el.id === +this.currentAnswerIndex).label; 
          this.text.find((elem) => elem.id === +this.currentDragId).answerId = this.answers.find((el) => el.id === +this.currentAnswerIndex).id;
          this.answers.find((elem) => elem.id === +this.currentAnswerIndex).disabled = true;
          this.answers.find((elem) => elem.id === +this.currentAnswerIndex).selected = false;
          this.currentAnswerIndex = null;
          this.currentDragId = null;
        } else {
          this.answers.find((elem) => elem.id === +this.currentAnswerIndex).selected = false;
        }
      }
    },
  },
};
</script>

<style scoped>
.body {
  width: 100%;
  height: 100vh;
  background-color: #f8f8fa;
}
.wrapper {
  max-width: 1110px;
  margin: 0 auto;
}
.description {
  display: flex;
  align-items: center;
  padding-top: 40px;
  margin: 0 40px 0 40px;
  color: #4c4c4c;
}
.info{
  margin-right: 15px;
}
.content {
  margin-right: 40px;
  margin-left: 40px;
  margin-top: 40px;
  background-color: #ffffff;
  border-radius: 5px;
  box-shadow: rgba(99, 99, 99, 0.2) 0px 2px 8px 0px;
}
.text {
  line-height: 55px;
  font-size: 18px;
  padding: 25px;
  align-items: center;
  align-content: center;
}
.answers {
  border-top: 2px dashed #bbbacc;
  padding: 40px 25px 40px 25px;
  display: grid;
  align-items: center;
  grid-template-columns: repeat(4, 1fr);
  justify-items: center;
  gap: 20px;
}
.answer-btn {
  max-width: 160px;
  width: 100%;
  border-radius: 10px;
  background: #1b998b;
  color: #ffffff;
  font-weight: 600;
  height: 40px;
}
.no-selected {
  width: 150px;
  border: 1px solid #d9d9d9;
  border-radius: 10px;
  margin-right: 15px;
  color: #1b998b;
  height: 36px;
}
.selected-answer {
  background-color: #1b998b;
  opacity: 0.5;
}
.disabled {
  background: #d3edec;
  color: #1b998b;
  cursor: not-allowed;
  border: 1px solid #1b998b;
  opacity: 0.5;
}
.selected {
  background: #1b998b;
  color: #ffffff;
  border: none;
  position: relative;
}
.cancel {
  position: absolute;
  right: -11px;
  top: -9px;
  background: #ffffff;
  border-radius: 50px;
}
.text-items{
  line-height: 30px;
}
.text-item{
  margin-right: 15px;
}
</style>
