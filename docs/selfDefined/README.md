

## 模板解析
### 模板数据
```
  <!-- 目前可用数据 -->

        idKey: "id",    //唯一标识
        address: "",    //模型控制器
        columns: [],    //teble 列
        search: [],     //搜索条件
        install: [],    //添加字段
        update: []      //修改字段
        pageButtons:{
            /** 添加按钮 */
            install: boolean,
            /** 添加修改 */
            update: boolean,
            /** 删除按钮 */
            delete: boolean,
            /** 导入按钮 */
            import: boolean,
            /** 导出按钮 */
            export: boolean
        }
```
### 解析方法
!> **模板解析数据 {{{ <解析方法> <数据Key> }}}**
```
    import { action, observable, runInAction, toJS } from "mobx";
    import { HttpBasics } from "core/HttpBasics";
    import { message } from "antd";
    import storeBasice from 'core/storeBasice';
    export class Store extends storeBasice {
        constructor() {
            super({
                // api 地址前缀
                address: '{{{ address }}}'
            });
        }
        /** 数据 ID 索引 */
        IdKey ='{{{ idKey }}}';
        /** table 列配置  title dataIndex 必备字段 其他为api 返回默认字段*/
        columns = {{{JSONColumns columns }}}
    }
    export default new Store();
```
!> **解析后的数据:**
```
  import { action, observable, runInAction, toJS } from "mobx";
    import { HttpBasics } from "core/HttpBasics";
    import { message } from "antd";
    import storeBasice from 'core/storeBasice';
    export class Store extends storeBasice {
        constructor() {
            super({
                // api 地址前缀
                address: '/corp/'
            });
        }
        /** 数据 ID 索引 */
        IdKey ='id';
        /** table 列配置  title dataIndex 必备字段 其他为api 返回默认字段*/
        columns = [
            {
                "title": "公司ID",
                "dataIndex": "id",
                "format": ""
            },
            ......
        ]
    }
    export default new Store();
```
### 自定义方法
!> **{{{JSONColumns columns }}} 中的 JSONColumns**
```
  // registerHelper 接受2个参数  解析函数名称 （JSONColumns） & 解析 方法回调 返回解析后数据
    // person 为 传入的 数据原型 （swagger 解析的 columns 字段数据）
    Handlebars.registerHelper('JSONColumns', function (person) {
        return JSON.stringify(person.filter(x => x.attribute.available).map(x => {
            return {
                title: x.description,
                dataIndex: x.key,
                format: x.format || '',
            }
        }), null, 4)
    });
```
!> 文件所在路径：
<img src="../_img/模板解析.png" width="150%">
## 功能重写
!> **所创组件的模板的store都继承至基类**

!> **继承的store支持重写但不能重载,且可以新建**

!> **重写方法时，请遵循TypeScript规范**
### 重写
!> **原方法:**
```javascript
async onDelete(params: any[]) {
    params = params.map(x => x[this.IdKey])
    const method = this.Urls.delete.method;
    const src = this.Urls.delete.src;
    const res = await this.Request[method](src, params).toPromise()
    if (res) {
      message.success('删除成功')
      this.onSelectChange([]);
      // 刷新数据
      this.onSearch();
    } else {
      message.success('删除失败')
    }
    return res
  }

```

!> **重写后:**
```javascript
 //重写删除
  @action.bound
  async onDelete(params: any[]) {
    const method = this.Urls.delete.method;
    const src = this.Urls.delete.src;
  //  const res = await this.Request[method](src, params).toPromise()
    if (true) {
      message.success('删除成功')
      this.onSelectChange([]);
      // 刷新数据
      //this.onSearch();
      //本地删除  
     runInAction(()=>{
       this.dataTT.list = this.filterData(params, this.dataTT.list)
      })
    } else {
      message.success('删除失败')
    }
    //return result
  }
```
### 新增
```javascript
 //新增操作
   @action.bound
   onAdd(record){
     if(record){
       this.record=record
     }else{
       this.addAll=true;
       this.record=this.dataTT.list.slice()
     }
     this.onModalShow({})
    
   }
   //修改操作
   @action.bound
   onUp(record){
     this.record=record
     this.onModalShow(record)
   }


```

## 结构重写

!> **组件结构继承至 "wtm/components/table"**

!> **框架依赖于React,且支持ant-design**

!> **继承的结构:**
```javascript
render() {
    const dataSource = this.Store.dataSource;
    /**
    * 行选择
    */
    const rowSelection = {
      selectedRowKeys: this.Store.selectedRowKeys,
      onChange: e => this.Store.onSelectChange(e),
    };
    const columns = this.Store.SwaggerModel.columns.slice();
    columns.push({
      title: 'Action',
      dataIndex: 'Action',
      render: this.renderAction.bind(this),
    })
    return (
      <Row ref={e => this.rowDom = ReactDOM.findDOMNode(e) as any}>
        <Divider />
        <Table
          bordered
          components={this.components}
          dataSource={dataSource.list.slice()}
          onChange={this.onChange.bind(this)}
          columns={columns}
          rowSelection={rowSelection}
          loading={this.Store.pageState.loading}
          pagination={
            {
              // hideOnSinglePage: true,//只有一页时是否隐藏分页器
              position: "top",
              showSizeChanger: true,//是否可以改变 pageSize
              pageSize: dataSource.pageSize,
              defaultPageSize: dataSource.pageSize,
              defaultCurrent: dataSource.pageNo,
              total: dataSource.count
            }
          }
        />
      </Row>
    );
  }

```

!> **重写后:**
```javascript
render(){
      console.log(this.initData(this.dataSource.slice()))
        const dataSource = this.Store.dataTT;

        const rowSelection = {
           selectedRowKeys: this.Store.selectedRowKeys,
            onChange: e => this.Store.onSelectChange(e),
          };
          const columns = this.Store.columns.slice();
          columns.push({
            title: 'Action',
            dataIndex: 'Action',
            render: this.renderAction.bind(this),
          })
          columns[0].width="30%"
        return (
            <Table
            expandedRowKeys={this.props.Store.expandedRowKeys.slice()}
            onExpand={this.props.Store.onExpand}
            bordered
            dataSource={dataSource.list.slice()}
            onChange={this.onChange.bind(this)}
            columns={columns}
            rowSelection={rowSelection}
            loading={this.Store.pageState.loading}
            // pagination={
            //   {
            //     // hideOnSinglePage: true,//只有一页时是否隐藏分页器
            //     position: "top",
            //     showSizeChanger: true,//是否可以改变 pageSize
            //     pageSize: dataSource.pageSize,
            //     defaultPageSize: dataSource.pageSize,
            //     defaultCurrent: dataSource.pageNo,
            //     total: dataSource.count
            //   }
            // }
          />
        )
    }

```

## 样式修改

!> **样式修改建议结合ant-design自带的类名来修改**

## 模板创建

!> **模板生成组件组件规则:框架会将模板的文件复制一份到"src/containers"里,其中带有模板语法的会解析,其余不变**

!> **模板中报错可忽略,模板的所有代码都只需考虑在"src/containers"中是否符合标准即可**

?> 模板编写目录:"wtmfront/template"

<img src="../_img/模板目录.png" width="30%">
