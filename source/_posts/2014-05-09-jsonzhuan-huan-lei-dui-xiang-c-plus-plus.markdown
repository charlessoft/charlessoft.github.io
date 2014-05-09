---
layout: post
title: "Json转换类对象C++"
date: 2014-05-09 13:49:11 +0800
comments: true
categories: 
---
#c++如何把json字符串转换成类对象(一)

##概述
c++中获取到json字符串后,每次需要遍历json字符串,然后把该json字符串中各个key对应的value取出来,赋值给各自的类对象.


##相关库
###1.jsoncpp
用于解析、遍历json库

##原理
根据json字符串,定义对应类,例如:

``` json
{
    "students": [
        {
            "name": "zhangsan",
            "age": 20,
            "sex": "男"
        },
        {
            "name": "lisi",
            "age": 21,
            "sex": "女"
        }
    ]
}

```

从以上json格式看,students节点包含2个数组,每个数组中都有一个name和age, 我们可以定义2个类, CStudents和CStudentItem类,
CStudents包含CStudentItem成员对象, CStudentItem包含name、age和sex成员函数

``` C++
class CStudentItem
{
private:
    string m_name;
    int m_age;
    string sex;
    
}

class CStudents
{
private:
    vector<CStudentItem*> m_StudentItemArr;
}
```

下一步 我们需要解析Json,把json中对象的值设置到以上对应的类对象中,我们就面临2个问题,
1.怎么遍历?可以从jsoncpp的demo中找到答案-见下面代码
2.遍历完怎么设置到对应对象中?使用函数指针.



###函数指针
事先定义好函数,在各自函数中赋值对象值,用来遍历json时候,调用该函数,进行赋值  
void Set_XXX( string strKey, void* value );  
其中XXX为自定义的函数名称,取个和json对象对应的名字, 例如: void Set_Name( string strKey, void* value );
代码修改成如下格式:


```

class CStudentItem
{
private:
    string m_name;
    int m_age;
    string sex;
   
 typedef void ( CStudentItem::*StudentItemFunc )( string key, void* value );

	 typedef map<string, StudentItemFunc>JsonMethodMap;

    void Set_Name( string strKey, void* value )
    {
        this->m_name = (char*)value;
    }

    void Set_Age( string strKey, void* value )
    {
        this->m_age = atoi((char*)value));
    }

    void Set_Sex( string strKey, void* value )
    {
        this->m_sex = (char*)value;
    }
}


class CStudents
{
private:
    vector<CStudentItem*> m_StudentItemArr;
}


```



``` c++

class IParseJson  
{  

public:
  
	IParseJson() {}   

	virtual ~IParseJson() {}

	virtual BOOL ParseJson( const char* json );

protected:

	virtual void DealJsonNode( string strNode, string value ){}

	virtual void DealJsonNode( string strNode, int value ){}

	virtual void DealJsonNode( string strNode, unsigned int value ){}

	virtual void DealJsonNode( string strNode, double value ){}

	virtual IParseJson* CreateJsonItem( string strKey );

private:
	virtual void PrintValueTree( Json::Value &value, IParseJson* pParent, IParseJson* pCurObj, const std::string strkey );


};

```




