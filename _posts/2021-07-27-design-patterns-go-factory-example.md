---
layout: post
title: Go语言工厂模式实例
date: 2021-07-27
tags: ["设计模式","设计模式"]
categories: 设计模式

---

## Go语言工厂模式实例
使用工厂模式完成一个案例：一个调度器

### 定义
```go
type Scheduler interface {
	Data(*FinalData) error //预处理数据
	Do() error //执行逻辑
	Output() error //输出逻辑
}
```
### 实现

```go
type FormatJsonData struct {
	req        string
	jsonStr    string
	res        bytes.Buffer
	outputPath string
}

// 获取实例方法
func NewFormatJsonClient() Tool {
	return new(FormatJsonData)
}

// 预处理逻辑需要的方法
func (d *FormatJsonData) Data(req *FinalData) error {
	if value, ok := req.Data.(string); ok {
		if req.Output != "" {
			d.outputPath = req.Output
		}
		if value[0] == '{' || value[0] == '[' {
			d.req = value
		} else if lib.FileExist(value) {
			d.req = lib.ReadFile(value)
		} else {
			return errors.New("request data is illegal")
		}
		return nil
	}
	return errors.New("request data is illegal")
}

// 逻辑实现
func (d *FormatJsonData) Do() error {
	return json.Indent(&d.res, []byte(d.req), "", "    ")
}

// 输出
func (d *FormatJsonData) Output() error {
	if d.outputPath != "" {
		content := d.res.Bytes()
		if err := ioutil.WriteFile(d.outputPath, content, 0666); err != nil {
			return err
		}
	} else {
		fmt.Println(d.res.String())
	}
	return nil
}
```
### 调用

```go
// 主函数
func main() {
	client := getClient()
	finalData := logics.FinalData{
		Data: Args.CommandData,
		Output: Args.CommandOutput,
	}
	if err := client.Data(&finalData); err != nil {
		log.Fatalln(err)
	}
	if err := client.Do(); err != nil {
		log.Fatalln(err)
	}
	if err := client.Output(); err != nil {
		log.Fatalln(err)
	}
}

//获取具体实例
func getClient() logics.Tool {
	var client logics.Tool
	switch Args.CommandName {
	case global.CommandFormatJson:
		client = logics.NewFormatJsonClient()
		break
	case global.CommandHelp:
		client = logics.NewHelpClient()
		break
	}
	if client == nil {
		log.Fatalln("client init failed")
	}
	return client
}

```