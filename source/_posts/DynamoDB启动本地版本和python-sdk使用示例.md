---
title: 本地DynamoDB的启动方法
date: 2018-01-18
categories: database
---
从http://dynamodb-local.s3-website-us-west-2.amazonaws.com/dynamodb_local_2016-05-17.tar.gz下载DynamoDB的按照包，本地jre环境的配置请自行配置
```
# tar zxf dynamodb_local_2016-05-17.tar.gz
# mkdir data
# java -Djava.library.path=./DynamoDBLocal_lib -jar DynamoDBLocal.jar -sharedDb -dbPath data
Initializing DynamoDB Local with the following configuration:
Port:	8000
InMemory:	false
DbPath:	data
SharedDb:	true
shouldDelayTransientStatuses:	false
CorsParams:	*
```
### Table 创建
```
# -*- coding:utf-8 -*-

import boto3
from boto3.dynamodb.conditions import Key, Attr

dynamodb = boto3.resource('dynamodb',
                          endpoint_url='http://localhost:8000')


def table_create():
    table = dynamodb.create_table(
        TableName='users',
        KeySchema=[
            {
                'AttributeName': 'username',
                'KeyType': 'HASH'
            },
            {
                'AttributeName': 'last_name',
                'KeyType': 'RANGE'
            }
        ],
        AttributeDefinitions=[
            {
                'AttributeName': 'username',
                'AttributeType': 'S'
            },
            {
                'AttributeName': 'last_name',
                'AttributeType': 'S'
            },

        ],
        ProvisionedThroughput={
            'ReadCapacityUnits': 5,
            'WriteCapacityUnits': 5
        }
    )
    table.meta.client.get_waiter('table_exists').wait(TableName='users')
```
### 获取Table
```
def table_get():
    return dynamodb.Table('users')
```
### 写入item
```
def item_put():
    table = table_get()
    table.put_item(
        Item={
                'username': 'liujinliu',
                'first_name': 'Jinliu',
                'last_name': 'liu',
                'age': 30,
                'account_type': 'standard_user',
            }
    )
```
### 获取item
```
def item_get():
    table = table_get()
    response = table.get_item(
        Key={
            'username': 'liujinliu',
            'last_name': 'liu',
        }
    )
    item = response['Item']
    return item
```
### update item
```
def item_update():
    table = table_get()
    table.update_item(
        Key={
            'username': 'liujinliu',
            'last_name': 'liu'
        },
        UpdateExpression='SET age = :val1',
        ExpressionAttributeValues={
            ':val1': 26
        }
    )
```
### 删除item
```
def item_delete():
    table = table_get()
    table.delete_item(
        Key={
            'username': 'liujinliu',
            'last_name': 'liu'
        }
    )
```
### 获取表内item数量
```
def table_size():
    table = table_get()
    return table.item_count
```
### 批量更新/插入item
```
items = [
    {
        'username': 'liujinliu',
        'last_name': 'liu',
        'first_name': 'jinliu',
        'age': 25,
        'address': {
            'road': '1 Jefferson Street',
            'city': 'LA',
            'state': 'CA',
            'zipcode': '90001'
        }
    },
    {
        'username': 'wangyiyang',
        'last_name': 'wang',
        'first_name': 'yiyang',
        'age': 26,
        'address': {
            'road': 'huilongguan',
            'city': 'Beijing',
            'state': 'CHINA',
            'zipcode': '082'
        }
    },
    {
        'username': 'chenwenquan',
        'last_name': 'chen',
        'first_name': 'wenquan',
        'age': 27,
        'address': {
            'road': 'jintailu',
            'city': 'henan',
            'state': 'JP',
            'zipcode': '222'
        }
    },
    {
        'username': 'dengliangju',
        'last_name': 'deng',
        'first_name': 'liangju',
        'age': 28,
        'address': {
            'road': 'qingnianlu',
            'city': 'chengdu',
            'state': 'India',
            'zipcode': '333'
        }
    },
]


def batch_write():
    table = table_get()
    with table.batch_writer(overwrite_by_pkeys=['username',
                                                'last_name']) as batch:
        for item in items:
            batch.put_item(Item=item)
```
### query, scan
```
def query():
    table = table_get()
    response = table.query(
        KeyConditionExpression=Key('username').eq('wangyiyang')
    )
    items = response['Items']
    return items


def scan():
    table = table_get()
    response = table.scan(
        FilterExpression=Attr('age').lt(28)
    )
    items = response['Items']
    return items


def scan_1():
    table = table_get()
    response = table.scan(
        FilterExpression=Attr('age').lt(28) & Attr(
            'address.city').begins_with('B')
    )
    items = response.get('Items', [])
    return items
```
### 删除Table
```
def table_delete():
    table = table_get()
    table.delete()
```
### 测试
```
if __name__ == '__main__':
    table_create()
    # item_put()
    # print(item_get())
    # item_update()
    # print(item_get())
    # print(table_size())
    # item_delete()
    print(table_size())
    batch_write()
    print(table_size())
    # print(item_get())
    print(query())
    print(scan())
    print(scan_1())
    # table_delete()
    # print(query())
```
