<template>
    <div id="app">
        <div class="flat-form-wrapper">
            <form id="addForm" class="default-form flat-form" v-on:submit.prevent="addNewRow()">
                <p>
                    <label>{{columns[1]}}</label>
                    <input v-model="newData.name" name="addFormName" type="text">
                </p>
                <p>
                    <label>{{columns[2]}}</label>
                    <input v-model="newData.surname" name="addFormSurname" type="text">
                </p>
                <p>
                    <label>{{columns[3]}}</label>
                    <input v-model="newData.age" name="addFormAge" type="text">
                </p>
                <p>
                    <label>{{columns[4]}}</label>
                    <input v-model="newData.groupId" name="addFormGroupId" type="text">
                </p>
                <p>
                    <input type="submit" value="Добавить">
                </p>
            </form>
            <ul class="errors">
                <li v-show="!validation.name">Поле ФИО не может быть пустым.</li>
                <li v-show="!validation.surname">Введите корректный e-mail.</li>
                <li v-show="!validation.age">Поле Телефон не может быть пустым.</li>
                <li v-show="!validation.groupId">Поле Телефон не может быть пустым.</li>
            </ul>
        </div>
        <table>
            <thead>
            <tr>
                <th v-for="key in columns">
                    {{ key }}
                </th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="(item, index) in items">
                <td><input type="checkbox" :value="index" v-model="checkedList"></td>
                <td>{{item.name}}</td>
                <td>{{item.surname}}</td>
                <td>{{item.age}}</td>
                <td>{{item.groupId}}</td>
                <td><input type="button" value="Изменить" v-on:click="showModal(index)"/></td>
            </tr>
            </tbody>
        </table>
        <div class="bottom-panel">
            <input type="button" value="Удалить записи" v-on:click="removeRows()"/>
            {{checkedList}}
        </div>
        <modal v-if="showModalF" @close="closeModal()" @remove="removeRow()" @update="updateRow()">
            <div slot="body">
                <form id="modalForm" class="default-form modal-form">
                    <p>
                        <label>{{columns[0]}}</label>
                        <input name="modalFormId" type="hidden" v-model="currentData.id" v-bind:readonly="true">
                        <span>{{currentData.id}}</span>
                    </p>
                    <p>
                        <label>{{columns[1]}}</label>
                        <input name="modalFormName" type="text" v-model="currentData.name">
                    </p>
                    <p>
                        <label>{{columns[2]}}</label>
                        <input name="modalFormSurname" type="text" v-model="currentData.surname">
                    </p>
                    <p>
                        <label>{{columns[3]}}</label>
                        <input name="modalFormAge" type="text" v-model="currentData.age">
                    </p>
                    <p>
                        <label>{{columns[4]}}</label>
                        <input name="modalFormGroupId" type="text" v-model="currentData.groupId">
                    </p>
                </form>
                <ul class="errors">
                    <li v-show="!validation.name">Поле ФИО не может быть пустым.</li>
                    <li v-show="!validation.surname">Введите корректный e-mail.</li>
                    <li v-show="!validation.age">Поле Телефон не может быть пустым.</li>
                    <li v-show="!validation.groupId">Поле Телефон не может быть пустым.</li>
                </ul>
            </div>
        </modal>
    </div>
</template>

<script>

    import appData from '../appdata.json';

    export default {
        name: 'app',
        data() {
            return {
                showModalF: false,
                currentRow: '',
                currentData: '',
                newData: {},
                items: {},
                checkedList: [],
                columns: [
                    '', 'Имя', 'Фамилия', 'Возраст', 'Группа', ''
                ]
            }
        },
        components: {
            'modal': {
                template: '#modal-template'
            }
        },
        filters: {
            capitalize: function (str) {
                return str.charAt(0).toUpperCase() + str.slice(1)
            }
        },
        computed: {
            validation: function () {
                let validData = (!this.showModalF) ? this.newData : this.currentData;
                return {
                    name: !!validData.name ? validData.name.trim() : false,
                    surname: !!validData.surname ? validData.surname.trim() : false,
                    // email: emailRE.test(validData.email),
                    age: !!validData.age ? validData.age.trim() : false,
                    groupId: !!validData.groupId ? validData.groupId.trim() : false
                }
            },
            isValid: function () {
                let validation = this.validation;
                return Object.keys(validation).every(function (key) {
                    return validation[key]
                })
            }
        },
        methods: {
            getData: function (data) {
                if (!localStorage.getItem('studentsbook')) {
                    localStorage.setItem('studentsbook', JSON.stringify(data));
                }
                this.items = JSON.parse(localStorage.getItem('studentsbook'));
            },
            showModal: function (index) {
                this.currentData = this.items[index];
                this.currentRow = index;
                this.showModalF = true;
            },
            removeRow: function () {
                this.items.splice(this.currentRow, 1);
                localStorage.setItem('studentsbook', JSON.stringify(this.items));
                this.closeModal();
            },
            removeRows: function () {
                for (let i in this.checkedList) {
                    this.items.splice(this.checkedList[i], 1);
                }
                localStorage.setItem('studentsbook', JSON.stringify(this.items));
            },
            checkRow: function (index) {
                if (this.checkedList.indexOf(index) < 0)
                    this.checkedList.push(index);
                else
                    this.checkedList.splice(index, 1);
            },
            updateRow: function () {
                if (this.isValid) {
                    this.items[this.currentRow] = this.currentData;
                    localStorage.setItem('studentsbook', JSON.stringify(this.items));
                    this.closeModal();
                }
            },
            addNewRow: function () {
                if (this.isValid) {
                    this.newData.id = this.items[this.items.length - 1].id + 1;
                    this.items.push(this.newData);
                    localStorage.setItem('studentsbook', JSON.stringify(this.items));
                    this.newData = {}
                }
            },
            closeModal: function () {
                this.showModalF = false;
            }
        },
        created: function () {
            this.getData(appData);
        }
    }
</script>

<style>

</style>
