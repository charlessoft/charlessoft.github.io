---
layout: post
title: "把json字符串转换成C++类对象(二)"
date: 2014-05-10 01:25:15 +0800
comments: true
categories: C++
---

##概述
上一篇文章中讲述了在C++中如何把json字符串转换成c++类对象,[把json字符串转换成C++类对象(一)](http://charlessoft.github.io/blog/2014/05/09/jsonzhuan-huan-lei-dui-xiang-c-plus-plus/)其中使用到了开源库(jsoncpp),函数指针,事先需要声明每个类相关代码,遍历完毕json字符串就得到对应类对象

<!--more-->
事先声明的代码

```

typedef void ( CStudentItem::*StudentItemFunc )( string key, void* value );

typedef map<string, StudentItemFunc>JsonMethodMap;

JsonMethodMap m_jsonMethodMap;


void Set_XXX( string strKey, void* value );

virtual void DealJsonNode( string strNode, string value );

virtual void DealJsonNode( string strNode, int value );

```

当json字符串很多的时候.就造成每次都要声明重复的代码,过程很繁琐,而且容易写错,如何解决这个问题,或者可以相对简便些?


我想到了用宏定义来声明函数类似MFC中插入DECLARE_DYNAMIC等宏的方式,

我们把重复的代码提取出来,变成宏定义,插入到类声明中

宏定义
```
#define JSON_TYPE_STRING 1
#define JSON_TYPE_INT 2
#define JSON_TYPE_DOUBLE 3


#define JSON_DEFINE_METHODMAP( theclass ) \
    typedef void ( theclass::*theclass##Func )( string strNode, void* value ); \
        typedef map<string, theclass##Func>JsonMethodMap; \
            JsonMethodMap m_jsonmapfunc;


#define JSON_DEAL_NODE_STRING() \
    virtual void DealJsonNode( string strNode, string value ){ \
        JsonMethodMap::iterator Iter = m_jsonmapfunc.find( strNode ); \
        if ( Iter != m_jsonmapfunc.end() ) \
        { \
            (this->*m_jsonmapfunc[strNode])( strNode,  (void*)value.c_str() ); \
        } \
    }

#define JSON_DEAL_NODE_INT() \
    virtual void DealJsonNode( string strNode, int value ){ \
        JsonMethodMap::iterator Iter = m_jsonmapfunc.find( strNode ); \
        if ( Iter != m_jsonmapfunc.end() ) \
        { \
            (this->*m_jsonmapfunc[strNode])( strNode,  (void*)value ); \
        } \
    }

```

修改后的代码

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

        //定义 JsonMethodMap
        DECLARE_DYNAMIC( CStudentItem )

        JSON_TYPE_STRING()

        JSON_TYPE_INT()
        //定义 JsonMethodMap

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

        virtual IParseJson* CreateJsonItem( string strKey )
        {
            return this; //该类中没有一些数组,直接返回自身就好
        }


};

```

剩下的每个类就只需要定义Set_XXX进行设置就好了. 后期看看怎么把Set_XXX函数进行优化.




