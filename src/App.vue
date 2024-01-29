<template>
  <n-p>
    You have selected {{ checkedRowKeys.length }} row{{
      checkedRowKeys.length < 2 ? '' : 's'
    }}.
  </n-p>
  <div>
          <n-button @click="sortName">Sort </n-button>
  </div>

  <n-data-table
    ref="dataTableInst"
    :columns="columns"
    :data="data"
    :pagination="pagination"
    :row-key="rowKey"
    @update:checked-row-keys="handleCheck"
  />
</template>

<script>
import { defineComponent, ref , h, nextTick, computed } from "vue";
import { NDataTable ,NInput , NButton } from 'naive-ui'


const ShowOrEdit = defineComponent({
  props: {
    value: [String, Number],
    onUpdateValue: [Function, Array]
  },
  setup (props) {
    const isEdit = ref(false)
    const inputRef = ref(null)
    const inputValue = ref(props.value)
    function handleOnClick () {
      isEdit.value = true
      nextTick(() => {
        inputRef.value.focus()
      })
    }
    function handleChange () {
      props.onUpdateValue(inputValue.value)
      isEdit.value = false
    }
    return () =>
      h(
        'div',
        {
          style: 'min-height: 22px',
          onClick: handleOnClick
        },
        isEdit.value
          ? h(NInput, {
            ref: inputRef,
            value: inputValue.value,
            onUpdateValue: (v) => {
              inputValue.value = v
            },
            onChange: handleChange,
            onBlur: handleChange
          })
          : props.value
      )
  }
})

const createData = () => {
  return[
  {
    key: 0,
    name: 'John Brown',
    age: 32,
    address: 'New York No. 1 Lake Park',
    chinese: 98,
    math: 60,
    english: 70
  },
  {
    key: 1,
    name: 'Jim Green',
    age: 42,
    address: 'London No. 1 Lake Park',
    chinese: 98,
    math: 66,
    english: 89
  },
  {
    key: 2,
    name: 'Joe Black',
    age: 32,
    address: 'Sidney No. 1 Lake Park',
    chinese: 98,
    math: 66,
    english: 89
  },
  {
    key: 3,
    name: 'Jim Red',
    age: 32,
    address: 'London No. 2 Lake Park',
    chinese: 88,
    math: 99,
    english: 89
  }
]

}

export default defineComponent({
  setup() {
    const data = ref(createData())

    const checkedRowKeysRef = ref([]);

    const getDataIndex = (key) => {
      return data.value.findIndex((item) => item.key === key)
    }
    const page = ref(1)

    const handlePageChange = (curPage) => {
      page.value = curPage
    }

    const paginationRef = computed(() => ({
      pageSize: 10,
      page: page.value
    }))

    const dataTableInstRef = ref(null)

    return {
      data,
      checkedRowKeys: checkedRowKeysRef,
      pagination: {
        pageSize: 10
      },
      dataTableInst: dataTableInstRef,
      sortName () {
        dataTableInstRef.value.sort('name', 'ascend')
      },

      

      paginationRef,
      handlePageChange,
      columns: [
        {
          type: "selection",
          disabled(row) {
            return row.name === "Edward King 3";
          }
        },
        {
          title: 'Name',
          key: 'name',
          sorter: (row1, row2) => row1.name - row2.name,
          width: 150,
          render (row) {
            const index = getDataIndex(row.key)
            return h(ShowOrEdit, {
              value: row.name,
              onUpdateValue (v) {
                data.value[index].name = v
              }
            })
          }
        },
        {
          title: 'Age',
          key: 'age',
              sorter: (row1, row2) => row1.age - row2.age,
          width: 100,
          render (row) {
            const index = getDataIndex(row.key)
            return h(ShowOrEdit, {
              value: row.age,
              onUpdateValue (v) {
                data.value[index].age = v
              }
            })
          }
        },
        {
          title: 'Address',
          key: 'address',
          render (row) {
            const index = getDataIndex(row.key)
            return h(ShowOrEdit, {
              value: row.address,
              onUpdateValue (v) {
                data.value[index].address = v
              }
            })
          }
        }
      ],

      rowKey: (row) => row.address,
      handleCheck(rowKeys) {
        checkedRowKeysRef.value = rowKeys;
      },


    };
  },
  components: {
    NDataTable,
    NButton
  }
});
</script>