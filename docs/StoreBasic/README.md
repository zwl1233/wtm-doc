
### 页面动作

?> 控制页面的弹出框的显示隐藏(编辑框、导入导出框、数据加载的loading、数据提交的loading)


```javascript
//页面状态
@observable pageState = {
    visibleEdit: false,//编辑显示
    visiblePort: false,//导入显示
    loading: false,//数据加载
    loadingEdit: false,//数据提交
    isUpdate: false//编辑状态
  }

//根据不同的type 修改页面状态
  @action.bound
  onPageState(key: "visibleEdit" | "visiblePort" | "loading" | "loadingEdit" | "isUpdate", value?: boolean) {
    const prevVal = this.pageState[key];
    if (prevVal == value) {
      return
    }
    if (typeof value == "undefined") {
      value = !prevVal;
    }
    this.pageState[key] = value;
  }

// 编辑框显示  增加/修改
async onModalShow(details = {}) {
    if (details[this.IdKey] == null) {
      this.onPageState("isUpdate", false)
    } else {
      this.onPageState("isUpdate", true)
      details = await this.onDetails(details)
    }
    runInAction(() => {
      this.details = details
    })
    this.onPageState("visibleEdit", true)
  }
```

### 功能

?> 对表单的增、删、改、查、文件的导入、导出进行封装

#### 增加 & 修改

```javascript
async onEdit(params) {
    if (this.pageState.loadingEdit) {
      return
    }
    const details = { ...this.details, ...params }
    this.onPageState("loadingEdit", true)
    // 添加 | 修改
    if (this.pageState.isUpdate) {
      return await this.onUpdate(details)
    }
    return await this.onInsert(details)
  }

  // 添加
  async onInsert(params) {
    const method = this.Urls.insert.method;
    const src = this.Urls.insert.src;
    const res = await this.Request[method](src, params).toPromise()
    if (res) {
      message.success('添加成功')
      // 刷新数据
      this.onSearch()
      this.onPageState("visibleEdit", false)
    } else {
      message.error('添加失败')
    }
    this.onPageState("loadingEdit", false)
    return res
  }
  //修改
  async onUpdate(params) {
    const method = this.Urls.update.method;
    const src = this.Urls.update.src;
    const res = await this.Request[method](src, params).toPromise()
    if (res) {
      message.success('更新成功')
      // 刷新数据
      this.onSearch()
      this.onPageState("visibleEdit", false)
    } else {
      message.error('更新失败')
    }
    this.onPageState("loadingEdit", false)
    return res
  }
```

#### 删除

```javascript
//删除 
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


#### 查询

```javascript
//查询
async onSearch(params: any = {}, page: any = { pageNo: 1, pageSize: 10 }) {
    if (this.pageState.loading == true) {
      return message.warn('数据正在加载中')
    }
    this.onPageState("loading", true);
    this.searchParams = { ...this.searchParams, ...params };
    params = {
      ...page,
      search: this.searchParams
    }
    const method = this.Urls.search.method;
    const src = this.Urls.search.src;
    const res = await this.Request[method](src, params).map(data => {
      if (data.list) {
        data.list = data.list.map((x, i) => {
          // antd table 列表属性需要一个唯一key
          return { key: i, ...x }
        })
      }
      return data
    }).toPromise()
    runInAction(() => {
      this.dataSource = res || this.dataSource
      this.onPageState("loading", false)
    })
    return res
  }

```

#### 导入 & 导出

```javascript

@computed
  get importConfig() {
    const action = this.Request.address + this.Urls.import.src
    return {
      name: 'file',
      multiple: true,
      action: action,
      onChange: info => {
        const status = info.file.status
        // NProgress.start();
        if (status !== 'uploading') {
          console.log(info.file, info.fileList)
        }
        if (status === 'done') {
          const response = info.file.response
          // NProgress.done();
          if (response.status == 200) {
            // 刷新数据
            this.onSearch();
            message.success(`${info.file.name} file uploaded successfully.`)
          } else {
            message.error(`${info.file.name} ${response.message}`)
          }
        } else if (status === 'error') {
          message.error(`${info.file.name} file upload failed.`)
        }
      }
    }
  }

  async onExport(params = this.searchParams) {
    await this.Request.download({
      url: this.Request.address + this.Urls.export.src,
      body: params
    })
  }

  async onTemplate() {
    await this.Request.download({
      url: this.Request.address + this.Urls.template.src,
    })
  }

```
