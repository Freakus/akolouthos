<template>
  <div class="center">
      <span v-if="warning">Warning - The following characters have no equivalent and have been skipped: {{ invalidCharacters }}</span>
    </div>
    <div class="center">
    <span>
        <div><button @click="saveFile(false)" style="width:100%;">Save</button></div>
        <div><button @click="saveFile(true)" style="width:100%;">Save (No SVG Symbols)</button></div>
        <div><button @click="squareRows" style="width:100%;">Square(ish)</button></div>
        <div><label for="rows" style="float:left;">Rows:</label><input id="rows" type="number" v-model="rows" min="1" max="999999999" size="8" style="float:right;" /></div>
        <div><label for="scale" style="float:left;">Width:</label><input id="scale" type="number" v-model="width" min="1" max="999999999" size="8" style="float:right;" /></div>
    </span>
    <textarea v-model="text" rows="7" cols="100">{{ text }}</textarea>
  </div>
  <div :style="SVGStyle" v-html="SVG"></div>
</template>

<script>
import { objectToString } from '@vue/shared';
import glyphs from '/src/data/paths.json'

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
      text: "The music draws us but it wont save you until you add yourself to its song"
    }
  },

  computed: {
    knownGlyphs() {
      return Object.keys(glyphs).join("").replace(/[.*+?^${}()|[\]\\-]/g, "\\$&").replaceAll("\n", "\\n");
    },
    regexValid() {
      return new RegExp(`[${this.knownGlyphs}]`, "g");
    },
    regexInvalid() {
      return new RegExp(`[^${this.knownGlyphs}]`, "g");
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
    SVGPaths() {
      var paths = ""
      for (let index = 0; index < this.validCharacters.length; index++) {
          paths += `<svg x="${Math.floor(index / this.rows) * 2}" y="${(index % this.rows) * 2}">${this.generatePath(this.validCharacters[index])}</svg>`;
      }
      return paths;
    },
    SVG() {
        return `<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="100%" viewBox="${this.viewBox}">\n<defs>\n${this.SVGSymbols}</defs>\n<g>\n${this.SVGUses}</g>\n</svg>`
    },
    SVGNoSymbols() {
        return `<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="100%" viewBox="${this.viewBox}">\n<g>\n${this.SVGPaths}</g>\n</svg>`
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
      this.rows = Math.ceil(Math.sqrt(this.length));
    },

    generateSymbol(character) {
      if (glyphs[character])
        return `<symbol id="${glyphs[character]["name"]}" width="2" height="2" viewBox="0 0 2 2">${this.generatePath(character)}</symbol>\n`
      else
        return ""
    },

    generatePath(character) {
      if (glyphs[character])
        return `<path d="${glyphs[character]["path"]}" />\n`
      else
        return ""
    },

    mapCharacterToSVGUse(character, index) {
      return `<use href="#${glyphs[character]["name"]}" x="${Math.floor(index / this.rows) * 2}" y="${(index % this.rows) * 2}" style="opacity:1.0" fill="${this.color}" />\n`;
    },

    saveFile(noSymbols=false) {
        const link = document.createElement("a");
        var blob
        if (noSymbols)
            blob = new Blob([this.SVGNoSymbols], {type: 'image/svg+xml'});
        else
            blob = new Blob([this.SVG], {type: 'image/svg+xml'});
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