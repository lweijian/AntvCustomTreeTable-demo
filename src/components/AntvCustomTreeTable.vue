<template>
  <div class="home">
    <a-table
        ref="table"
        :columns="columns"
        :data-source="tabData"
        expandRowByClick
        :row-selection="{selectedRowKeys: selectedRowKeys, onSelectAll: onSelectAll, onSelect: onSelect}"
    />
  </div>
</template>

<script>
const columns = [
  {
    title: 'Name',
    dataIndex: 'name',
    key: 'name',
  },
  {
    title: 'Age',
    dataIndex: 'age',
    key: 'age',
    width: '12%',
  },
  {
    title: 'Address',
    dataIndex: 'address',
    width: '30%',
    key: 'address',
  },
];
const data = [
  {
    key: 1,
    name: 'John Brown sr.',
    age: 60,
    address: 'New York No. 1 Lake Park',
    children: [
      {
        key: 11,
        name: 'John Brown',
        age: 42,

        address: 'New York No. 2 Lake Park',
      },
      {
        key: 12,
        name: 'John Brown jr.',
        age: 30,
        address: 'New York No. 3 Lake Park',

        children: [
          {
            key: 121,
            name: 'Jimmy Brown',
            age: 16,
            address: 'New York No. 3 Lake Park',
          },
        ],
      },
      {
        key: 13,
        name: 'Jim Green sr.',
        age: 72,
        address: 'London No. 1 Lake Park',
        children: [
          {
            key: 131,
            name: 'Jim Green',
            age: 42,
            address: 'London No. 2 Lake Park',
            children: [
              {
                key: 1311,
                name: 'Jim Green jr.',
                age: 25,
                address: 'London No. 3 Lake Park',
              },
              {
                key: 1312,
                name: 'Jimmy Green sr.',
                age: 18,
                address: 'London No. 4 Lake Park',
              },
            ],
          },
        ],
      },
    ],
  },
  {
    key: 2,
    name: 'Joe Black',
    age: 32,
    address: 'Sidney No. 1 Lake Park',
  },
];
let flatMap=new Map()
function flatToMap(arr=[],map,parentKey=null) {
  if(!arr||arr.length===0) return []
  arr.forEach(item=>{
    item.parentKey=parentKey
    map.set(item.key,item)
    if(item?.children&&item?.children.length>0){
      flatToMap(item.children,map,item.key)
    }
  })
}
flatToMap(data,flatMap)


let checkedId=[]

function selectChiledren(key) {
  let node=flatMap.get(key)
  let res=[]
  res.push(key)
  node.isChecked=true
  node.halfChoice=false

  if(node?.children&&node?.children.length>0){
    node.children.forEach(item=>{
      if( checkedId.indexOf(item.key)===-1){
        res=[...res,...selectChiledren(item.key)]

      }
    })
  }

  return res
}
function unSelectChiledren(key) {
  let node=flatMap.get(key)
  checkedId=checkedId.filter(item=>(item!==key))

  node.isChecked=false
  node.halfChoice=false

  if(node?.children&&node?.children.length>0){
    node.children.forEach(item=>{
      if( checkedId.indexOf(item.key)!==-1){
        unSelectChiledren(item.key)
      }
    })
  }
}
function findParentNodeAll(key) {
  let node=flatMap.get(key)
  if(node?.parentArr){
    return node.parentArr
  }
  let  parentArr=[];
  if(node?.parentKey){
    let parentNode=flatMap.get(node?.parentKey)
    parentArr=[...parentArr,parentNode,...findParentNodeAll(node?.parentKey)]
  }
  node.parentArr=parentArr
  return parentArr
}

function isChlidAllChecked(key) {
  let node=flatMap.get(key)
  if(node?.children&&node?.children.length>0){
    return  node.children.every(item=>{
      return  checkedId.indexOf(item.key)>=0
    })
  }
  return  true;
}
function isChlidSomeChecked(key) {
  if(key===undefined) return  false
  let node=flatMap.get(key)
  if(node?.children&&node?.children.length>0){
    for(let child of node.children ){
      if(checkedId.indexOf(child.key)!==-1){
        return true
      }else {
        let isChildCheck= isChlidSomeChecked(child.key)
        if(isChildCheck) return  true
      }
    }

  }
  return  false;
}


function select(key) {
  checkedId=[...checkedId,...selectChiledren(key)]
  handleParentClick(key)
}
function unSelect(key) {
  unSelectChiledren(key)
  handleParentClick(key)
}
function  handleParentClick(key){
  let parentArr=findParentNodeAll(key)
  parentArr.forEach( parent=>{
    // 子元素全选了
    if(isChlidAllChecked(parent.key)){
      checkedId.push(parent.key)
      parent.isChecked=true
      parent.halfChoice=false
      //子元素没全选，但有选
    }else if( isChlidSomeChecked(parent.key)){
      parent.isChecked=false
      parent.halfChoice=true
      if(checkedId.indexOf(parent.key)!==-1){
        checkedId=checkedId.filter(key=>key!==parent.key)
      }
      // 子元素没有选
    }else {
      parent.isChecked=false
      parent.halfChoice=false
      if(checkedId.indexOf(parent.key)!==-1){
        checkedId=checkedId.filter(key=>key!==parent.key)
      }
    }
  })}

function click(key,selected) {

  if(selected){
    select(key)
  }else {
    unSelect(key)
  }
}
export default {
  name:'AntvCustomTreeTable',
  data() {
    return {
      selectedRowKeys:[],
      columns:columns,

      tabData: data,
    }
  },

  methods: {
    onSelectAll(selected) {
      if (selected) {
        const tabData = this.tabData;
        const arr = [];
        setVal(tabData, arr);
        this.selectedRowKeys = arr;
      } else {
        this.selectedRowKeys = [];
      }
      function setVal(list, arr) {
        list.forEach(v => {
          arr.push(v.key);
          if (v.children) {
            setVal(v.children, arr);
          }
        });
      }
    },

    onSelect(record, selected) {
      click(record.key,selected)
    for(let node of flatMap.values()){
      const el = this.$refs.table.$el.querySelector(`[data-row-key="${node.key}"] .ant-table-selection-column .ant-checkbox`)
      if(!el) continue
      if(node?.halfChoice){
        el.className=`${el.className} ant-checkbox-indeterminate`
      }else {
        el.className=el.className.replace('ant-checkbox-indeterminate','')
      }
    }
      this.selectedRowKeys=[...checkedId]
    },


  }
}

</script>
