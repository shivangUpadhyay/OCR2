<template>
    <div>
      <h1>Image Text Extraction</h1>
      <button @click="extractText">Extract Text</button>
      <div v-if="textExtracted">
        <h2>Extracted Text:</h2>
        <pre>{{ extractedText }}</pre>
        <canvas ref="wordFrequencyChart"></canvas>
      </div>
    </div>
  </template>
  
  <script>
  import { createWorker } from 'tesseract.js';
  import { Bar } from 'vue-chartjs';
  
  export default {
    data() {
      return {
        extractedText: '',
        textExtracted: false,
        wordFrequencyData: {
          labels: [],
          datasets: [
            {
              label: 'Word Frequency',
              backgroundColor: 'rgba(75, 192, 192, 0.2)',
              borderColor: 'rgba(75, 192, 192, 1)',
              borderWidth: 1,
              data: [],
            },
          ],
        },
      };
    },
    mixins: [Bar],
    methods: {
      async extractText() {
        const worker = createWorker();
        await worker.load();
        await worker.loadLanguage('eng');
        await worker.initialize('eng');
  
        await worker.recognize('imagetext.png', 'eng', function (logger) {
          console.log(logger);
        }).then(async function (result) {
          this.extractedText = result.data.text;
          this.textExtracted = true;
          worker.terminate();
  
          
          this.calculateWordFrequency();
        }.bind(this));
      },
      calculateWordFrequency() {
        const text = this.extractedText.toLowerCase();
        const words = text.split(/\s+/);
        const wordCount = {};
  
        for (const word of words) {
          if (wordCount[word]) {
            wordCount[word]++;
          } else {
            wordCount[word] = 1;
          }
        }
  
        const sortedWords = Object.keys(wordCount).sort((a, b) => wordCount[b] - wordCount[a]);
        const topWords = sortedWords.slice(0, 10);
  
        this.wordFrequencyData.labels = topWords;
        this.wordFrequencyData.datasets[0].data = topWords.map(word => wordCount[word]);
  
        
        this.renderChart(this.wordFrequencyData, { responsive: true });
      },
    },
  };
  </script>
  