const tableComponent = Vue.component('table-component', {
    template:`
    <div class="row">
        <div class="input-field col s6">
            <input placeholder="Filtrar" id="buscador" class="validate" type="text" v-model="filter" v-on:keyup="filterArray()">
            <label for="buscador" class="active">Filtrar</label>
        </div>
        <div class="col s12">
            <table class="">
                <thead>
                    <tr>
                        <th v-for="(header, header_index) in headerTable"> {{ header }} </th>
                        <th v-if="actions"> {{ actionHeaders }} </th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(body, body_index) in bodyTableRender">
                        <td v-for="(att, attr_index) in attrDisplay"> {{ displayAttr(body,att) }} </td>
                        <td v-if="actions">
                            <button v-for="btn in buttons" :class="btn.class" v-on:click="btnEvents(btn.action, body )"> {{btn.icon}} {{ btn.labelButton }} </button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div class="col s12">
            <ul class="pagination">
                <li :class="currentPage==1?'hidden_li':''" v-on:click="currentPage = currentPage-1">   
                    <a href="#!"><i class="mdi-navigation-chevron-left"></i></a>
                </li>

                <li v-for="(pag, pages_index) in pages" :class="currentPage==pag?'active':''" v-on:click="changePage(pag)"> <a href="#!">{{pag}} </a></li>
                
                <li :class="currentPage==lastPage || pages.length == 1?'hidden_li':''" v-on:click="currentPage = currentPage+1"><a href="#!">
                    <i class="mdi-navigation-chevron-right"></i></a>
                </li>
            </ul>
        </div>
    </div>
    `,
    data:function(){
        return {
            currentPage:1,
            lastPage:'',
            pagesArray:[],
            itemsTotal:[],
            filter:''
        }
    },
    props:{
        /**
         * Array que contiene las cabecera de la tabla
         * ejemplo:
         *  ['Nombre','Apellido','Direccion']
         */
        headerTable:{
            type: Array,
            required: true
        },
        /**
         * Array que contiene los elementos que tendra el cuerpo de la tabla (tr y td)
         * [{
         *  nombre:'asda',
         * apellido:'asd',
         * }]
         */
        bodyTable: {
            type: Array,
            required: true
        },
        /**
         * Cantidad de registros por pagina
         */
        forPage:{
            type:Number,
            default:6
        },
        /**
         * Array con los nombres de los atributos a consultar en el objeto json en orden 
         * para mostrar en cada columna de la tabla
         * 
         * ejemplo: 
         * ['nombre','apellido','...']
         */
        attrDisplay:{
            type:Array,
            required:true
        },
        /**
         * Varible para activar la columna de las opciones
         */
        actions:{
            type:Boolean,
            default:true
        },

        /**
         * Etiqueta para la columna de las opciones
         */
        actionHeaders:{
            type:String,
            default:'Opciones'
        },

        /**
         * Array de acciones que tendra la tabla
         * Ejemplo:
         * [{
            labelButton:'prueba', ->opcional
            action:'message', ->requerido
            icon:'', ->opcional
            class:'' ->opcional   
         * }]
         * 
         * Estos atributos no pueden cambiar.
         */
        buttons:{
            type:Array,
            required:false
        }
    },
    created:function(){
        this.itemsTotal = this.bodyTable;
    },
    computed: {
        /**
         * Muestro la pagina actual
         */
        isActive:function(val){
            if(val == this.currentPage){
                return true;
            }
        },

        /**
         * Muestro la cantidad de paginas posibles
         */
        pages:function(){
            let totalItems = this.itemsTotal.length;
            if(totalItems == 0){
                this.pagesArray.push(1)
                return this.pagesArray;
            }else{
                this.pagesArray= [];
                this.lastPage = Math.round(totalItems/this.forPage)
                if(this.lastPage==0){
                    this.pagesArray.push(1)
                }else{
                    for (let index = 1; index <= this.lastPage; index++) {
                        this.pagesArray.push(index)
                    }
                }
            }
            return this.pagesArray;
        },

        /**
         * Muestro la cantidad de registros por paginas
         */
        bodyTableRender:function(){
            if(this.filter==''){
                this.itemsTotal = this.bodyTable; 
            }

            if(this.currentPage==1){
                return this.itemsTotal.slice(0,this.forPage)
            }else{
                let since = parseInt(parseInt(this.currentPage)*parseInt(this.forPage))-this.forPage;
                let until = parseInt(parseInt(this.currentPage)*parseInt(this.forPage));
                return this.itemsTotal.slice(since,until)
            }
        }
    },
    methods:{
        /**
         * Función que muestra la información en la columna correspondiente
         */
        displayAttr:function(body, attr){
            return body[attr];
        },

        /**
         * Función que realiza el cambio de página
         */
        changePage:function(page){
            this.currentPage = page;
        },

        /**
         * Metodo que se emit el evento de los botones
         */
        btnEvents:function(param,obj){
            this.$emit(param, obj)
        },

        /**
         * Metodo para filtrar los resultados
         */
        filterArray:function(){
            let result = [];
            let result_tr = this.bodyTable;

            result_tr.forEach(element => {
                this.attrDisplay.forEach(element_attr => {
                    if(element[element_attr].toLowerCase().indexOf(this.filter.toLowerCase()) >= 0){
                        result.push(element);
                    }
                });
            });
            this.itemsTotal = result;
        }
    }
})