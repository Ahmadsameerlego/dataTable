<template>
   <div class="card">
    <div class="form-group position-relative">
          <InputText v-model="filters['global'].value" class="tableSearch" placeholder="يمكنك البحث " />
    </div>

    <div>
        <MultiSelect :modelValue="selectedColumns" :options="columns" optionLabel="header" @update:modelValue="onToggle"
          display="chip" placeholder="Select Columns" />
    </div>

    <div>
        <Button type="button" icon="pi pi-filter-slash" label="Clear" outlined @click="clearFilter()" />
    </div>

    <div>
        <Button type="button" label="Sort" outlined @click="sort()" />
    </div>
    <div>

    </div>
      <DataTable 
          :value="products"
          tableStyle="min-width: 50rem"
          paginator :rows="20"  
          :rowsPerPageOptions="[5, 10, 20, 50]" 
          sortMode="multiple"
          v-model:filters="filters"
          showGridlines 
          stripedRows 
          editMode="cell"
          v-model:selection="selectedProduct"
          :selectAll="selectAll"
          :totalRecords="totalRecords"
          @row-select="onRowSelect"
           @row-unselect="onRowUnselect"
          @select-all-change="onSelectAllChange"
          @cell-edit-complete="onCellEditComplete"
          dataKey="id" 
          ref="dt"
          selectionMode="single" :metaKeySelection="metaKey" 
          :removableSort="sorted"
        >
          <Column selectionMode="multiple" headerStyle="width: 3rem"></Column>
          <!-- <Column field="phone" sortable  header="phone"></Column>
          <Column field="name" sortable  header="Name"></Column>
          <Column field="email" sortable  header="email"></Column>
          <Column field="status" sortable  header="status"></Column> -->
          <Column v-for="col of selectedColumns" :key="col.field" :field="col.field" :header="col.header" sortable style="width: 25%">
              <template #body="{ data, field }">
                  {{ field === 'price' ? formatCurrency(data[field]) : data[field] }}
              </template>
              <template #editor="{ data, field }">
                  <template v-if="field !== 'price'">
                      <InputText v-model="data[field]" autofocus />
                  </template>
                  <template v-else>
                      <InputNumber v-model="data[field]" mode="currency" currency="USD" locale="en-US" autofocus />
                  </template>
              </template>
          </Column>
          
      </DataTable>

        <Button type="button"  label="addItem" outlined @click="addNewRow()" />
  </div>
</template>

<script>
import DataTable from 'primevue/datatable';
import Column from 'primevue/column';
import { FilterMatchMode } from 'primevue/api';
import InputText from 'primevue/inputtext';

// import ColumnGroup from 'primevue/columngroup';   // optional
// import Row from 'primevue/row';                   // optional
import { ProductService } from '@/service/ProdcutService';
import MultiSelect from 'primevue/multiselect';
import Button from 'primevue/button';
export default ({
  data() {
    return {
      products: null,
      selectedProduct: null,
      metaKey: true ,
      filters: {
            global: { value: null, matchMode: FilterMatchMode.CONTAINS },
      },
      selectAll: false,
      totalRecords: 0,

      columns: [
            { field: 'email', header: 'Code' },
            { field: 'name', header: 'Name' },
            { field: 'phone', header: 'Quantity' },
            { field: 'status', header: 'Price' }
        ]
      ,
      selectedColumns: null,
      sorted : false
    }   
  }, 
  methods: {
    addNewRow() {
      console.log(this.$refs.dt)
      this.$refs.dt.value.push({

      })
    },  
    clearFilter() {
        this.initFilters();
    },
    initFilters() {
        this.filters = {
            global: { value: null, matchMode: FilterMatchMode.CONTAINS },
        };
    },
    sort() {
      this.sorted = !this.sorted;
    },  

    onToggle(value) {
          this.selectedColumns = this.columns.filter(col => value.includes(col));
      },
    onSelectAllChange(event) {
          const selectAll = event.checked;

          if (selectAll) {
              ProductService.getProductsMini().then(data => {
                  this.selectAll = true;
                  this.selectedProduct = data.customers;
              });
          }
          else {
              this.selectAll = false;
              this.selectedProduct = [];
          }
    },
    onRowSelect() {
      this.selectAll = this.selectedProduct.length === this.totalRecords
    },
    onRowUnselect() {
      this.selectAll = false;
    },
    onCellEditComplete(event) {
            let { data, newValue, field } = event;

            switch (field) {
                case 'quantity':
                case 'price':
                    if (this.isPositiveInteger(newValue)) data[field] = newValue;
                    else event.preventDefault();
                    break;

                default:
                    if (newValue.trim().length > 0) data[field] = newValue;
                    else event.preventDefault();
                    break;
            }
        },
  },
    

  mounted() {
    ProductService.getProductsMini().then((data) => (this.products = data));
    this.selectedColumns = this.columns;

  }, 
  components: {
    DataTable,
    Column,
    InputText,
    MultiSelect,
    Button
    // Row,
    // ColumnGroup
    
  }
});
</script>