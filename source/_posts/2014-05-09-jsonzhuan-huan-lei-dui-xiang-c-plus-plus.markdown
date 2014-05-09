---
layout: post
title: "把json字符串转换成C++类对象(一)"
date: 2014-05-09 13:49:11 +0800
comments: true
categories: c++
---

##概述
c++中获取到json字符串后,每次需要遍历json字符串,然后把该json字符串中各个key对应的value取出来,赋值给各自的类对象.


##相关库
###1. jsoncpp
用于解析、遍历json库

##原理
###1. 遍历json
###2. 函数指针赋值

根据json字符串,定义对应类,例如:

``` 
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

从以上json格式看,`students`节点包含2个数组,每个数组中都有一个`name`和`age`, 我们可以定义2个类, `CStudents`和`CStudentItem`类,
`CStudents`包含`CStudentItem`成员对象, `CStudentItem`包含`name`、`age`和`sex`成员函数

``` 
class CStudentItem
{
private:
    CStudentItem(){}

    virtual ~CStudentItem(){}

    string m_name;

    int_ptr_t m_age;

    string sex;
    
};

class CStudents
{
private:
    vector<CStudentItem*> m_StudentItemArr;
};

```

下一步 我们需要解析Json,把json中对象的值设置到以上对应的类对象中,我们就面临2个问题,  
1. 怎么遍历?可以从jsoncpp的demo中找到答案-见下面代码  
2. 遍历完怎么设置到对应对象中?使用函数指针.  

####遍历json
建立一个IParseJson类,用于解析遍历Json  
IParseJson.h
``` 

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

`PrintValueTree` 遍历json,在遍历中处理各自结点信息  
`DealJsonNode` 在函数中,参数strNode是传递进来的结点名字,查找map对应的函数指针,进行赋值  
`CreateJsonItem` 在函数中,参数strKey针对传递进来的结点名字,进行判断处理是否生成员变量对象.  


IParseJson.cpp
```


#include <algorithm> // sort
#include <stdio.h>
#include <stddef.h>
#include <sstream>
IParseJson* IParseJson::CreateJsonItem( string strKey )
{
    return this;
}

void IParseJson::PrintValueTree( Json::Value &value, IParseJson* pParent, IParseJson* pobj, const std::string strkey )
{
    IParseJson* pTmpJsonNode = NULL;
    switch ( value.type() )
    {
        case Json::nullValue:
            {
                break;
            }
        case Json::intValue:
            pobj->DealJsonNode( strkey, value.asInt() );
            break;
        case Json::uintValue:
            pobj->DealJsonNode( strkey, value.asUInt() );
            break;
        case Json::realValue:
            pobj->DealJsonNode( strkey, value.asDouble() );
            break;
        case Json::stringValue:
            pobj->DealJsonNode( strkey, value.asString() );
            break;
        case Json::booleanValue:
            pobj->DealJsonNode( strkey, value.asBool() );
            break;
        case Json::arrayValue:
            {
                int size = value.size();

                for ( int index =0; index < size; ++index )
                {
                    pTmpJsonNode = pParent->CreateJsonItem( strkey );
                    PrintValueTree( value[index], pParent, pTmpJsonNode, strkey );
                }

            }
            break;
        case Json::objectValue:
            {
                Json::Value::Members members( value.getMemberNames() );
                std::sort( members.begin(), members.end() );
                std::string suffix = "";
                for ( Json::Value::Members::iterator it = members.begin();
                        it != members.end();
                        ++it )
                {
                    const std::string &name = *it;

                    PrintValueTree( value[name], pobj, pobj, name );

                }
            }
            break;
        default:
            break;
    }
}



BOOL IParseJson::ParseJson( const char* json )
{

    Json::Features features;
    Json::Reader reader( features );
    Json::Value root;
    bool parsingSuccessful = reader.parse( json, root );
    if ( !parsingSuccessful )
    {
        return FALSE;
    }

    PrintValueTree( root, this, this , "" );
    return TRUE;
}


``` 



####函数指针
事先定义好函数,在各自函数中赋值对象值,用来遍历json时候,调用该函数,进行赋值  
void Set_XXX( string strKey, void* value );  
其中XXX为自定义的函数名称,取一个和json对象对应的名字, 例如: void Set_Name( string strKey, void* value );
代码修改成如下格式:


```

class CStudentItem
{
private:
    string m_name;
    
    int m_age;
    
    string sex;
   
    //设置函数指针
    CStudentItem()
    {
        //设置结点对应的函数指针
        m_jsonMethonMap["name"] = &CStudentItem::Set_Name;
        m_jsonMethonMap["age"] = &CStudentItem::Set_Age;
        m_jsonMethonMap["sex"] = &CStudentItem::Set_Sex;
    }

    virtual ~CStudentItem(){}

    typedef void ( CStudentItem::*StudentItemFunc )( string key, void* value );

    typedef map<string, StudentItemFunc>JsonMethodMap;

    JsonMethodMap m_jsonMethonMap;

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
    //设置函数指针
};


class CStudents
{
private:
    vector<CStudentItem*> m_StudentItemArr;
};


```

现在实现了遍历Json,定义好函数指针后,怎么赋值?  
我们让所有类都继承IParseJson, 实现DealJsonNode 函数,就可以实现对个类对象进行赋值  
修改后的代码:

```

class CStudentItem : public IParseJson
{
    public:
        string m_name;

        int m_age;

        string m_sex;
        //设置函数指针
        CStudentItem()
        {
            //设置结点对应的函数指针
            m_jsonMethodMap["name"] = &CStudentItem::Set_Name;
            m_jsonMethodMap["age"] = &CStudentItem::Set_Age;
            m_jsonMethodMap["sex"] = &CStudentItem::Set_Sex;
        }

        virtual ~CStudentItem(){}

        typedef void ( CStudentItem::*StudentItemFunc )( string key, void* value );

        typedef map<string, StudentItemFunc>JsonMethodMap;

        JsonMethodMap m_jsonMethodMap;

        void Set_Name( string strKey, void* value )
        {
            this->m_name = (char*)value;
        }

        void Set_Age( string strKey, void* value )
        {
            this->m_age = (intptr_t)value;
        }

        void Set_Sex( string strKey, void* value )
        {
            this->m_sex = (char*)value;
        }
        //设置函数指针


        //在map中找到对应的函数指针,对成员变量进行赋值
        virtual void DealJsonNode( string strNode, string value )
        {
            JsonMethodMap::iterator Iter = m_jsonMethodMap.find( strNode );
            if ( Iter != m_jsonMethodMap.end() )
            {
                (this->*m_jsonMethodMap[strNode])( strNode,  (void*)value.c_str() );
            }
        }

        virtual void DealJsonNode( string strNode, int value )
        {
            JsonMethodMap::iterator Iter = m_jsonMethodMap.find( strNode );
            if ( Iter != m_jsonMethodMap.end() )
            {
                (this->*m_jsonMethodMap[strNode])( strNode,  (void*)value );
            }
        }
        //在map中找到对应的函数指针,对成员变量进行赋值

        virtual IParseJson* CreateJsonItem( string strKey )
        {
            return this; //该类中没有一些数组,直接返回自身就好
        }


};


```

调用

main.cpp

```
#include <stdio.h>
#include <stdlib.h>
#include <string>
using namespace std;

#pragma comment(lib,"./jsoncpp/dbg/lib_json/json_vc10_libmtd.lib")

int main()
{

	string strJson = "{ \"students\": [ { \"name\": \"zhangsan\", \"age\": 20, \"sex\": \"男\" }, { \"name\": \"lisi\", \"age\": 21, \"sex\": \"女\" } ] }";
	CStudents* pStudents = new CStudents;
	pStudents->ParseJson( strJson.c_str() );
	for( int i=0;i<pStudents->m_StudentItemArr.size(); i++ )
	{
		CStudentItem* pStudentItem = pStudents->m_StudentItemArr.at(i);
		printf("name:%s\r\nage:%d\r\nsex:%s\r\n-----\r\n",pStudentItem->m_name.c_str(),
			pStudentItem->m_age,pStudentItem->m_sex.c_str() );
	}
}
```

结果:

{% img /../images/json_to_obj/1.png %}  


小结:

只要声明好各个类成员变量并设置好对应的函数,就可以把json字符串转换成对应的类对象,
每个类对象都需要定义JsonMethodMap来保存函数指针,对于少量的json字符串,还可以接受,如果json字符串比较多,并且包含多个json数组等,我们就需要声明多个类,并且每个类都需要处理JsonMethodMap,比较繁琐,而且容易出错.

如何解决这个问题?可以使用一些宏定义,类似MFC中的定义一些列宏的思想来解决. 下文介绍

[json2obj demo](http://url.cn/KmkwHO)


