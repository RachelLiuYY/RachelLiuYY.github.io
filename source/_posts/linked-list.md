---
title: linked-list
date: 2021-06-24 11:11:36
tags: [data structure]
---

## 链表结点结构
```
struct Node
{
    int data;
    Node *next;
};
typedef struct Node Node;
```

## 常用功能

### 链表逆序
```
Node * ReverseList(Node *head)
{
    Node *p = head->next;
    Node *q = head->next->next;
    Node *t = NULL;
    while(q! = NULL)
    {
        t = q->next;
        q->next = p;
        p = q;
        q = t;
    }
    head->next->next = NULL;
    head->next = p;
    return head;
}
```

### 链表合并
```
// 已知两个链表head1和head2各自有序，合并为一个依然有序的链表

Node * Merge(Node *head1 , Node *head2)
{
if ( head1 == NULL)
    return head2 ;
if ( head2 == NULL)
    return head1 ;
Node *head = NULL ;
Node *p1 = NULL;
Node *p2 = NULL;
if ( head1->data < head2->data )
{
    head = head1 ;
    p1 = head1->next;
    p2 = head2 ;
}
else
{
    head = head2 ;
    p2 = head2->next ;
    p1 = head1 ;
}
Node *pcurrent = head ;
while ( p1 != NULL && p2 != NULL)
{
       if ( p1->data <= p2->data )
       {
            pcurrent->next = p1 ;
            pcurrent = p1 ;
            p1 = p1->next ;
       }
       else
       {
            pcurrent->next = p2 ;
            pcurrent = p2 ;
            p2 = p2->next ;
       }
}
if ( p1 != NULL )
    pcurrent->next = p1 ;
if ( p2 != NULL )
    pcurrent->next = p2 ;
return head ;
}
```
## 链表合并(递归)
```
Node * MergeRecursive(Node *head1 , Node *head2)
{
    if ( head1 == NULL )
        return head2 ;
    if ( head2 == NULL)
        return head1 ;
    Node *head = NULL ;
    if ( head1->data < head2->data )
    {
        head = head1 ;
        head->next = MergeRecursive(head1->next,head2);
    }
    else
    {
        head = head2 ;
        head->next = MergeRecursive(head1,head2->next);
    }
    return head ;
}
```