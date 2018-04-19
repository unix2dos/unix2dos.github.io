---
title: 'c++stl容器循环earse用法'
tags: c++
---

### vector deque
在使用 vector、deque遍历删除元素时，也可以通过erase的返回值来获取下一个元素的位置：

```
      std::vector< int> Vec;
      std::vector< int>::iterator itVec;
      for( itVec = Vec.begin(); itVec != Vec.end(); )
      {
            if( WillDelete( *itVec) )
            {
                 itVec = Vec.erase( itVec);
            }
            else
               itList++;
      }
```
<!-- more -->

### list set map 
在 使用 list、set 或 map遍历删除某些元素时可以这样使用：

```
      std::list< int> List;
      std::list< int>::iterator itList;
      for( itList = List.begin(); itList != List.end(); )
      {
            if( WillDelete( *itList) )
            {
               itList = List.erase( itList);
            }
            else
               itList++;
      }
```


```
      std::list< int> List;
      std::list< int>::iterator itList;
      for( itList = List.begin(); itList != List.end(); )
      {
            if( WillDelete( *itList) )
            {
               List.erase( itList++);
            }
            else
               itList++;
      }
```
