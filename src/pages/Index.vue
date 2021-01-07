<template>
  <q-page class="fle flex-center">
    <div class="column">
      <div class="row q-ma-md">
        <q-input
          class="full-width"
          v-model="jsonText"
          outlined
          color="primary"
          type="textarea"
          placeholder="Paste your JSON here"
        ></q-input>
      </div>
      <div class="row">
        <q-btn
          class="q-ma-md"
          color="primary"
          label="Convert"
          @click="convertJSON"
        />
        <q-btn
          class="q-ma-md"
          color="primary"
          label="Reset"
          @click="resetFields"
        />
        <q-btn
          :disable="!showDownload"
          class="q-ma-md"
          color="primary"
          label="Download CSV"
          @click="downloadCSV"
        />
      </div>
      <div class="row q-ma-md">
        <div class="text-red">{{ errors }}</div>
      </div>
      <div class="row q-ma-md">
        <q-input
          class="full-width"
          v-model="csvText"
          outlined
          color="primary"
          type="textarea"
          placeholder="CSV Result"
        ></q-input>
      </div>
    </div>
  </q-page>
</template>

<script>
export default {
  name: 'PageIndex',
  data () {
    return {
      jsonText: '',
      csvText: '',
      errors: '',
      showDownload: false
    }
  },
  methods: {
    validateContent () {
      if (this.jsonText === '') {
        this.errors = 'Campo JSON vazio. Insera um JSON válido para converter.'
        return false
      }
      return true
    },
    convertJSON () {
      if (this.validateContent()) {
        try {
          var json = JSON.parse(this.jsonText)
          var fields = Object.keys(json[0])
          var replacer = function (key, value) {
            return value === null ? '' : value
          }
          var csv = json.map(function (row) {
            return (
              '"' +
              fields
                .map(function (fieldName) {
                  return JSON.stringify(row[fieldName], replacer)
                })
                .join('";')
                .replace(/"/g, '"')
            )
          })
          csv.unshift('"' + fields.join('";"') + '"') // add header column
          csv = csv.join('\r\n')
          this.csvText = csv
          this.showDownload = true
        } catch {
          this.errors =
            'Não foi possível converter o JSON. Verifique se o conteúdo informado é um JSON válido.'
        }
      }
    },
    downloadCSV () {
      var element = document.createElement('a')
      element.setAttribute(
        'href',
        'data:text/csv;charset=utf-8,' + encodeURIComponent(this.csvText)
      )
      element.setAttribute('download', 'table.csv')

      element.style.display = 'none'
      document.body.appendChild(element)

      element.click()

      document.body.removeChild(element)
    },
    resetFields () {
      this.jsonText = ''
      this.csvText = ''
      this.errors = ''
      this.showDownload = false
    }
  },
  watch: {
    jsonText () {
      if (this.jsonText.length > 0) {
        this.errors = ''
      }
    }
  }
}
</script>

<style>
textarea {
  min-height: 25vh !important;
}
</style>
