<template>
  <div
      class="calculator draggable"
      ref="calculator"
      @mousedown="startDrag"
      :style="{ top: position.y + 'px', left: position.x + 'px', position: 'absolute' }"
  >
    <div class="display">{{ displayValue }}</div>
    <div class="buttons">
      <button
          v-for="button in buttons"
          :key="button"
          @click="handleClick(button)"
          :class="{ operator: isOperator(button) }">
        {{ button }}
      </button>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      displayValue: '0',
      currentInput: '',
      buttons: [
        'C', '⌫', ' ', ' ', ' ',
        '7', '8', '9', '(', ')',
        '4', '5', '6', '×', '÷',
        '1', '2', '3', '√', '^',
        ',', '0', '=', '+', '-',
      ],
      isDragging: false,
      position: {x: 0, y: 0},
      offset: {x: 0, y: 0}
    };
  },
  mounted() {
    this.centerCalculator();
    window.addEventListener('resize', this.centerCalculator);
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.centerCalculator);
  },
  methods: {
    handleClick(button) {
      switch (button) {
        case 'C':
          this.clear();
          break;
        case '=':
          this.calculateInServer();
          break;
        case '⌫':
          this.deleteLast();
          break;
        case '√':
          this.checkOperatorBeforeBracket(button);
          break;
        case this.isOperator(button):
          this.addOperator(button);
          break;
        case ' ':
          break;
        default:
          this.addDigit(button);
      }
    },
    deleteLast() {
      if (this.currentInput.length > 0) {
        this.currentInput = this.currentInput.slice(0, -1);
        this.displayValue = this.currentInput || '0';
      }
    },
    clear() {
      this.displayValue = '0';
      this.currentInput = '';
    },
    async calculateInServer() {

      const sanitizedInput = this.currentInput
          .replace(/÷/g, '/')
          .replace(/×/g, '*')
          .replace(/√/g, 'r');

      await axios.post('https://localhost:7169/Calculate/calc', {
            expression: sanitizedInput,
          }
      )
          .then(response => {
                const res = parseFloat(response.data.toFixed(6));

                this.displayValue = String(res);
                this.currentInput = String(res);
              }
          )
          .catch(error => {
                console.log(error)
                this.displayValue = error.response.data.message;
                this.currentInput = '';
              }
          );
    },
    addOperator(operator) {
      const lastChar = this.currentInput.slice(-1);
      console.log(lastChar);

      if (this.isOperator(lastChar)) {
        this.currentInput = this.currentInput.slice(0, -1) + operator;
        console.log(this.currentInput.slice(-1));
      } else {
        this.currentInput += operator;
      }
      this.displayValue = this.currentInput;
    },
    addDigit(digit) {
      if (this.displayValue === '0' && digit !== '.') {
        this.displayValue = digit;
        this.currentInput = digit;
      } else {
        this.currentInput += digit;
        this.displayValue = this.currentInput;
      }
    },
    isOperator(value) {
      return ['÷', '×', '-', '+', '^', '√', '(', ')'].includes(value);
    },
    startDrag(event) {
      this.isDragging = true;
      this.offset.x = event.clientX - this.position.x;
      this.offset.y = event.clientY - this.position.y;
      window.addEventListener('mousemove', this.drag);
      window.addEventListener('mouseup', this.stopDrag);
    },
    drag(event) {
      if (this.isDragging) {
        this.position.x = event.clientX - this.offset.x;
        this.position.y = event.clientY - this.offset.y;
      }
    },
    stopDrag() {
      this.isDragging = false;
      window.removeEventListener('mousemove', this.drag);
      window.removeEventListener('mouseup', this.stopDrag);
    },
    centerCalculator() {
      const centerX = window.innerWidth / 2;
      const centerY = window.innerHeight / 2;
      this.position = {
        x: centerX - 110,
        y: centerY - 200
      };
    },
    checkOperatorBeforeBracket(operator) {
      const lastChar = this.currentInput.slice(-1);
      if (this.isOperator(lastChar)) {
        this.currentInput += operator;
        this.currentInput += '('
      } else if (this.displayValue === '0') {
        this.displayValue = operator;
        this.currentInput = operator;
        this.currentInput += '('
      }
      this.displayValue = this.currentInput;
    }
  }
};
</script>

<style>
.calculator {
  width: 260px;
  margin: 50px auto;
  font-family: Arial, sans-serif;
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 10px;
  background: #f9f9f9;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.display {
  height: 50px;
  background: #222;
  color: #fff;
  text-align: right;
  padding: 10px;
  font-size: 1.5em;
  border-radius: 4px;
  margin-bottom: 10px;
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.3);
}

.buttons {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 10px;
}

button {
  height: 50px;
  font-size: 1.2em;
  border: none;
  background: #e0e0e0;
  color: #333;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.2s;
}

button.operator {
  background: #f9a825;
  color: white;
}

button:hover {
  background: #d6d6d6;
}

button.operator:hover {
  background: #e68a00;
}
</style>
