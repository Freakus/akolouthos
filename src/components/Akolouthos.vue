<template>
  <div class="center">
      <span v-if="warning">Warning - The following characters have no equivalent and have been skipped: {{ invalidCharacters }}</span>
    </div>
    <div class="center">
    <span>
        <div><button @click="saveFile" style="width:100%;">Save</button></div>
        <div><button @click="squareRows" style="width:100%;">Square(ish)</button></div>
        <div><label for="rows">Rows:</label><input id="rows" type="number" v-model="rows" min="1" max="999999999" size="8"/></div>
        <div><label for="scale">Width:</label><input id="scale" type="number" v-model="width" min="1" max="999999999" size="8"/></div>
    </span>
    <textarea v-model="text" rows="5" cols="100">{{ text }}</textarea>
  </div>
  <div :style="SVGStyle" v-html="SVG"></div>
</template>

<script>
import { objectToString } from '@vue/shared';

export default {
  // Properties returned from data() become reactive state
  // and will be exposed on `this`.
  props: {
    color: {
        type: String,
        default(props) {
            return "#000000";
        }
    }
  },

  data() {
    return {
      rows: 1,
      width: 600,
      // Message to be transliterated.
      text: "The music draws us but it wont save you until you add yourself to its song",
      // SVG paths for the various glyphs.
      glyphs: {
        "a": "M 1 0 L 2 1 L 2 2 L 1 1 L 0 2 L 0 1 Z",
        "b": "M 0 0 L 2 0 L 1 1 L 2 2 L 0 2 Z",
        "c": "M 0 1 L 1 0 L 2 0 L 1 1 L 2 2 L 1 2 Z",
        "d": "M 0 0 L 1 0 L 2 1 L 1 2 L 0 2 Z",
        "e": "M 0 0 L 2 0 L 2 1 L 1 2 L 0 2 Z",
        "f": "M 0 0 L 2 0 L 2 1 L 1 1 L 1 2 L 0 2 Z",
        "g": "M 1 0 L 2 0 L 2 2 L 0 2 L 1 1 Z",
        "h": "M 0 0 L 1 1 L 2 0 L 2 2 L 1 1 L 0 2 Z",
        "i": "M 0 0 L 1 0 L 1 2 L 0 2 Z",
        "j": "M 1 0 L 2 0 L 2 2 L 0 2 L 0 1 L 1 1 Z",
        "k": "M 1 0 L 2 0 L 1 1 L 2 2 L 0 2 L 0 1 Z",
        "l": "M 0 0 L 1 0 L 1 1 L 2 2 L 0 2 L 0 1 Z",
        "m": "M 0 0 L 2 0 L 2 2 L 1 1 L 0 2 Z",
        "n": "M 1 0 L 2 0 L 2 1 L 1 1 L 0 2 L 0 1 Z",
        "o": "M 0 0 L 2 0 L 2 2 L 0 2 Z",
        "p": "M 0 0 L 2 0 L 1 1 L 1 2 L 0 2 Z",
        "q": "M 0 0 L 2 0 L 2 1 L 1 1 L 2 2 L 0 2 Z",
        "r": "M 0 0 L 2 0 L 2 1 L 1 2 L 0 2 L 1 1 Z",
        "s": "M 1 0 L 2 1 L 1 2 L 0 2 L 0 1 Z",
        "t": "M 0 0 L 1 0 L 2 1 L 1 2 L 0 1 Z",
        "u": "M 0 0 L 2 0 L 2 2 L 1 2 L 0 1 Z",
        "v": "M 0 0 L 1 1 L 2 0 L 2 1 L 1 2 L 0 1 Z",
        "w": "M 0 0 L 2 0 L 2 1 L 1 2 L 0 1 Z",
        "x": "M 1 0 L 2 1 L 1 2 L 0 1 Z",
        "y": "M 1 0 L 2 1 L 2 2 L 0 2 L 0 1 Z",
        "z": "M 1 0 L 2 0 L 2 1 L 1 2 L 0 2 L 0 1 Z",
        " ": "M 0 1 L 2 1 L 2 2 L 0 2 Z",
      }
    }
  },

  computed: {
    knownGlyphs() {
      return Object.keys(this.glyphs).join("");
    },
    regexValid() {
        return new RegExp(`[${this.knownGlyphs}]`, "gm");
    },
    regexInvalid() {
        return new RegExp(`[^${this.knownGlyphs}]`, "gm");
    },
    // Returns true if there are unmapped characters in the message.
    warning() {
      return this.text.toLowerCase().replace(this.regexValid,"").length > 0;
    },
    // Returns a distinct list of the characters that couldn't be mapped.
    invalidCharacters() {
      return [...new Set(this.text.toLowerCase().replace(this.regexValid,"").split(""))].join("");
    },
    // Returns the message with any invalid characters removed.
    validCharacters() {
      return this.text.toLowerCase().replace(this.regexInvalid,"").split("");
    },
    // Returns a deduplicated list of the characters in the message.
    distinctCharacters() {
        return [...new Set(this.validCharacters)];
    },
    // Returns the length of the message with any invalid characters removed.
    length() {
      return this.validCharacters.length;
    },
    cols() {
      return Math.ceil(this.length / this.rows);
    },
    SVGSymbols() {
      var symbols = ""
      for (let index = 0; index < this.distinctCharacters.length; index++) {
        symbols += this.generateSymbol(this.distinctCharacters[index]);
      }
      return symbols;
    },
    SVGUses() {
      var uses = ""
      for (let index = 0; index < this.validCharacters.length; index++) {
          uses += this.mapCharacterToSVGUse(this.validCharacters[index], index);
      }
      return uses;
    },
    SVG() {
        return `<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="100%" viewBox="${this.viewBox}"><defs>${this.SVGSymbols}</defs><g>${this.SVGUses}</g></svg>`
    },
    viewBox() {
      return `0 0 ${ this.cols * 2 } ${ this.rows * 2 }`
    },
    svgWidth() {
      if (this.width)
        return this.width;
      else
        return this.rows * 50;
    },
    SVGStyle() {
        return `width: ${this.width}px;`;
    }
  },

  // Methods are functions that mutate state and trigger updates.
  // They can be bound as event listeners in templates.
  methods: {
    squareRows() {
      this.rows = Math.floor(Math.sqrt(this.length));
    },

    generateSymbol(character) {
      if (this.glyphs[character])
        return `<symbol id="${character}" width="2" height="2" viewBox="0 0 2 2"><path d="${this.glyphs[character]}"/></symbol>`
      else
        return ""
    },
    mapCharacterToSVGUse(character, index) {
      return `<use href="#${character}" x="${Math.floor(index / this.rows) * 2}" y="${(index % this.rows) * 2}" style="opacity:1.0" fill="${this.color}" />\n`;
    },

    saveFile() {
        const link = document.createElement("a");
        const blob = new Blob([this.SVG], {type: 'image/svg+xml'});
        link.setAttribute('href', URL.createObjectURL(blob));
        link.setAttribute('download', `${this.validCharacters.join("")}.svg`);
        link.click();
        URL.revokeObjectURL(link.href);
    }
  },

  beforeMount(){
    this.squareRows();
    this.width = this.rows * 100;
  }
}
</script>