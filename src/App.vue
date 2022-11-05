<template>
  <div>
    <div style="text-align: center">
      <h1>Classificar e extrair texto de imagens</h1>
      <label class="file" style="text-align: left">
        <input type="file" id="file" multiple @change="changeInputFile">
        <span class="file-custom"></span>
      </label>
    </div>
    <div style="text-align: center">
      <img v-if="loading" src="./imgs/loading.gif" style="margin: auto">
    </div>
    <div v-if="renderTables">
      <table v-for="(imagem, idx) in arrImagens" v-bind:key="idx">
        <thead>
          <tr>
            <th colspan="2" style="font-weight: bold">{{imagem.imagem.name}}</th>
          </tr>
        </thead>
        <tbody>
          <template v-if="!imagem.error">
            <tr>
              <td style="font-weight: bold">Classificação</td><td>{{imagem.getImageClass.class}}</td>
            </tr>
            <tr v-for="(text, key) in imagem.getImageText" v-bind:key="key">
              <td style="font-weight: bold">{{objTraducao[key]}}</td><td>{{text}}</td>
            </tr>
          </template>
          <tr v-else>
            <td colspan="2" style="font-weight: bold">{{imagem.error}}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

const BASE_URL = 'http://127.0.0.1:5000';

export default {
  name: 'App',
  components: {
    
  },
  data(){
    return{
      objTraducao: {
        "rg": "RG",
        "cpf": "CPF",
        "cnh": "CNH",
        "nome": "Nome",
        "cnh_validade": "Validade da CNH",
        "data_nascimento": "Data de nascimento",
        "naturalidade": "Naturalidade",
      },
      arrImagens: [],
      renderTables: false,
      loading: false,
    }
  },
  methods: {
    async changeInputFile(){
      this.arrImagens = [];
      this.renderTables = false;
      this.loading = true;
      const files = document.querySelector('input[type=file]').files;
      if(files.length === 0){
        alert("Selecione um arquivo"); return false;
      }
      
      for(let i = 0; i < files.length; i++){
        let file = files[i];
        this.arrImagens[i] = {imagem: file, getImageClass: undefined, getImageText: undefined, error: undefined};
        let base64File = await this.convertImageBase64(file);
        let dataClass = JSON.stringify({"img_base64": base64File.split(",")[1]})

        await axios.post(BASE_URL+'/getImageClass', dataClass, {headers: {'Content-Type': 'application/json'}}).then(response => {
          this.arrImagens[i].getImageClass = response.data;
        }).catch(error => {
          console.log(error)
          this.arrImagens[i].error = "Erro ao classificar imagem!";
        });

        if(this.arrImagens[i].getImageClass.confidence < 90){
          this.arrImagens[i].error = "Classificação da imagem não identificada!";
        }else{
          let dataText = JSON.stringify({"img_class": this.arrImagens[i].getImageClass.class,"img_base64": base64File.split(",")[1]})
  
          await axios.post(BASE_URL+'/getImageText', dataText, {headers: {'Content-Type': 'application/json'}}).then(response => {
            this.arrImagens[i].getImageText = response.data;
          }).catch(error => {
            console.log(error)
            this.arrImagens[i].error = "Erro ao extrair texto da imagem!";
          });
        }
      }
      this.loading = false;
      this.renderTables = true;
    },
    convertImageBase64(file){
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onloadend = () => {
          resolve(reader.result);
        }
        reader.onerror = reject;
        reader.readAsDataURL(file);
      });
      
    }
  }
}
</script>

<style>
  table {
    border: 1px solid #005E94;
    background-color: #C8EDFF;
    width: 100%;
    text-align: center;
    margin-top: 15px;
  }
  table td, table th {
    border: 1px solid #5B6394;
    padding: 4px 4px;
  }
  table tbody td {
    font-size: 13px;
  }
  table thead {
    background: #005E94;
    background: -moz-linear-gradient(top, #4086af 0%, #196e9e 66%, #005E94 100%);
    background: -webkit-linear-gradient(top, #4086af 0%, #196e9e 66%, #005E94 100%);
    background: linear-gradient(to bottom, #4086af 0%, #196e9e 66%, #005E94 100%);
  }
  table thead th {
    font-size: 17px;
    font-weight: bold;
    color: #F0F0F0;
    text-align: center;
    border-left: 2px solid #5B6394;
  }
  table thead th:first-child {
    border-left: none;
  }
  .file {
    position: relative;
    display: inline-block;
    cursor: pointer;
    height: 2.5rem;
  }
  label, input, select, button {
    font: 1rem/1.5 "Helvetica Neue", Arial, sans-serif;
  }
  .file input {
    min-width: 14rem;
    margin: 0;
    opacity: 0;
  }
  .file-custom {
    position: absolute;
    top: 0;
    right: 0;
    left: 0;
    z-index: 5;
    height: 2.5rem;
    padding: 0.5rem 1rem;
    line-height: 1.5;
    color: rgb(0, 0, 0);
    background-color: #C8EDFF;
    border: 0.075rem solid #005E94;
    border-radius: 0.25rem;
    box-shadow: inset 0 0.2rem 0.4rem rgb(0 0 0 / 5%);
    user-select: none;
  }
  .file-custom:after {
    content: "Escolher imagens";
  }
  .file-custom:before {
    position: absolute;
    top: -0.075rem;
    right: -0.075rem;
    bottom: -0.075rem;
    z-index: 6;
    display: block;
    content: "Procurar";
    height: 2.5rem;
    padding: 0.5rem 1rem;
    line-height: 1.5;
    color: rgb(255, 255, 255);
    background-color: #005E94;
    border: 0.075rem solid #005E94;
    border-radius: 0 0.25rem 0.25rem 0;
  }
  *, *:before, *:after {
    box-sizing: border-box;
  }
  body{
    background: #ebf8ff;
  }
</style>
