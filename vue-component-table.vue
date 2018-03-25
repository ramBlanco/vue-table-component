const tableComponent = Vue.component('table-component', {
    template:`
    <div class="row">
        <div class="">
            <label for="buscador" class="active">Filtrar</label>
            <input placeholder="Filtrar" id="buscador" class="validate" type="text" v-model="filter" v-on:keyup="filterArray()">
        </div>
        <div class="col s12">
            <table v-bind:class="classTable">
                <thead>
                    <tr>
                        <th v-for="(header, header_index) in headerTable" v-on:click="sortArray(attrDisplay[header_index])"> {{ header.nombre }} </th>
                        <th v-if="actions"> {{ actionHeaders }} </th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(body, body_index) in bodyTableRender">
                        <td v-for="(att, attr_index) in attrDisplay" 
                            v-on:click="exeEvent(headerTable[attr_index].event,body)" 
                            v-bind:class="headerTable[attr_index].event != '' ? 'class-event': ''" > {{ displayAttr(body,att) }} </td>
                        <td v-if="actions">
                            <button v-for="btn in buttons" v-bind:class="btn.class" v-on:click="exeEvent(btn.action, body )"> {{btn.icon}} {{ btn.labelButton }} </button>
                        </td>
                    </tr>
                    <tr v-if="bodyTableRender.length == 0">
                        <td vbind:colspan="headerTable.length"> No se han encontrado registros </td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div class="">
            <label for="" class="">Cantidad de registros por página:</label>
            <select class="browser-default" name="" id="" v-model="forPage">
                <option value="5">5</option>
                <option value="10">10</option>
                <option value="25">25</option>
                <option value="50">50</option>
                <option value="100">100</option>
            </select>
        </div>
        <div class="col s12">
            <ul v-bind:class="ulClassPaginate">
                <li v-bind:class="currentPage==1?'hidden_li':'page-item'" v-on:click="currentPage = currentPage-1">   
                    <a href="#!" class="page-link"><i :class="liPreview"></i></a>
                </li>

                <li v-for="(pag, pages_index) in pages" v-bind:class="currentPage==pag?'active page-item':'page-item'" v-on:click="changePage(pag)"> <a href="#!" class="page-link">{{pag}} </a></li>
                
                <li v-bind:class="currentPage==lastPage || pages.length == 1?'hidden_li page-item':'page-item'" v-on:click="currentPage = currentPage+1"><a href="#!" class="page-link">
                    <i v-bind:class="liNext"></i></a>
                </li>
            </ul>
        </div>
    </div>
    `,
    data:function(){
        return {
            currentPage:1,      // Pagina por defecto
            forPage:10,         // Cantidad de registros por página
            lastPage:'',        // Última página
            pagesArray:[],      // Array que contendra las paginas a mostrar
            itemsTotal:[],      // Array que contendra los datos
            filter:'',          // Variable que usara el buscador para filtrar los registros
            sortDescAsc:false   // Orden actual del array (descendiente o ascendiente)
        }
    },
    props:{
        /**
         *  Cadena de texto para las clases que tendra la tabla
         */
        classTable:{
            type:String,
            default:''
        },

        /**
         * Array que contiene las cabecera de la tabla
         * ejemplo:
         * 
         * [{nombre:'',event:''}]
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
         * Array con los nombres de los atributos a consultar en el objeto json en orden 
         * con la cabecera de la tabla para mostrar en cada columna
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
        },

        /**
         * Cadena de texto para la clase del ul paginador
         */
        ulClassPaginate:{
            type:String,
            default:'pagination'
        },

        /**
         * Cadena de texto para el li que regresa la página
         */
        liPreview:{
            type:String,
            default:'mdi-navigation-chevron-left'
        },

        /**
         * Cadena de texto para el li que avanza la página
         */
        liNext:{
            type:String,
            default:'mdi-navigation-chevron-right'
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
            this.pagesArray= [];
            let totalItems = this.itemsTotal.length;
            if(totalItems == 0){
                this.pagesArray.push(1)
                return this.pagesArray;
            }else{
                this.pagesArray= [];
                this.lastPage = Math.ceil((totalItems/this.forPage))
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
                this.setPos(0,this.forPage)
                return this.itemsTotal.slice(0,this.forPage)
            }else{
                let since = parseInt(parseInt(this.currentPage)*parseInt(this.forPage))-this.forPage;
                let until = parseInt(parseInt(this.currentPage)*parseInt(this.forPage));
                this.setPos(since,until)
                return this.itemsTotal.slice(since,until)
            }
        },
    },
    methods:{
        /**
         * Función que muestra la información en la columna correspondiente
         */
        displayAttr:function(obj, attr){
            if(attr.indexOf('.') == -1){
                return obj[attr];
            }else{
                return viewDataObject(obj, attr);
            }
        },
        /**
         * Funcion que retorna el valor de la posición cuando el objeto es bidimensional
         */
        viewDataObject:viewDataObject,

        /**
         * Función que realiza el cambio de página
         */
        changePage:function(page){
            this.currentPage = page;
        },

        /**
         * Metodo que emit el evento de los botones
         */
        exeEvent:function(param,obj){
            this.$emit(param, obj)
        },

        /**
         * Metodo para filtrar los resultados desde el buscador
         */
        filterArray:function(){
            let result = [];
            let result_tr = this.bodyTable;

            result_tr.forEach(element => {
                this.attrDisplay.forEach(element_attr => {
                    let search = element_attr.indexOf('.') == -1 ? element[element_attr].toLowerCase() : viewDataObject(element,element_attr)
                    if(search.indexOf(this.filter.toLowerCase()) >= 0){
                        result.push(element);
                    }
                });
            });
            this.itemsTotal = result;
        },

        /**
         *  Metodo para ordernar el array de datos
         */
        sortArray:function(sort){
            this.itemsTotal.sort( (a,b) => {
                if(this.sortDescAsc){
                    //descendiente
                    return a[sort] > b[sort] ? -1:1;
                }else{
                    //ascendiente
                    return a[sort] > b[sort] ? 1:-1;
                }
                return 0;
            })
            this.sortDescAsc = this.sortDescAsc == true ? false:true
        },

        /**
         *  Método para setear la posicion del objeto dentro del array
         */
        setPos:function(since, until){
            for (let index = since; index <= until; index++) {
                if(this.itemsTotal[index] != undefined){
                    this.itemsTotal[index].positionJson = index;
                }
            }
        }
    }
})

function viewDataObject(obj, attr){
    console.log(attr)
    var array_attr = attr.split('.')
    var data = '';
    for (let index = 0; index < array_attr.length; index++) {
        if(data==''){
            data = obj[array_attr[index]] != undefined ? obj[array_attr[index]] : '' ;
        }else{
            data = data[array_attr[index]] != undefined ? data[array_attr[index]] : '';
        }
    }
    return data;
}