---
date: "2019-05-26+08:00"
publishdate: "2019-05-26+08:00"
lastmod: "2019-05-28+08:00"
draft: false
title: "Java手写红黑树"
tags: ["修行", "Java", "数据结构"]
series: ["技术"]
categories: ["学习"]
toc: true
---
昨晚跟人置气，要手写一个红黑树。
事实证明第一次写还是很难得，用IDE都搞了大半天时间。
主要方法和难点基本都搞定了，先结个案，以后有空一定要手写出来。

```java
package com.ysc.may.models;

import java.util.concurrent.locks.ReentrantLock;

public class RedBlackTree {
    public RedBlackTree() {
    }

    public class Node {
        public boolean has = false;
        public int left = 0;
        public int right = 0;
        public int father = 0;
        public int value;

    }

    private Node[] arr = new Node[16];
    private int last = 0;// 表示数组已放置元素数量
    
    private ReentrantLock lock = new ReentrantLock();

    private void expand() {
        Node[] temp = arr;
        arr = new Node[2 * arr.length];
        for (int i = 0; i < temp.length; i++) {
            arr[i] = temp[i];
        }
        for (int i = arr.length / 2; i < arr.length; i++) {
            arr[i] = new Node();
        }
        balance(0);
    }

    public int search(int value, int index) {

        if (last <= 1) {
            return index;
        }
        if (arr[index].left != 0 && arr[arr[index].left].value >= value) {

            return search(value, arr[index].left);
        }
        if (arr[index].right != 0 && arr[arr[index].right].value < value) {

            return search(value, arr[index].right);
        }
        return index;

    }

    private int getBouble() {
        for (int i = 0; i < arr.length; i++) {
            if (!arr[i].has) {
                return i;
            }
        }
        return arr.length - 1;
    }

    private boolean putRight(int index, int value) {
        Node father = arr[index];
        if (value > father.value) {
            if (father.right == 0) {// 右节点为空，直接放置
                father.right = getBouble();
                arr[father.right].has = true;
                arr[father.right].value = value;
                arr[father.right].father = index;
                if (father.left == 0 && (arr[father.father].right==0)){
                    cZip(father.father);
                } else if (arr[father.father].right == 0 && father.right == 0 && last > 2) {
                    rorateRight(father.father, 2);

                }
            } else {// 该子树满，左链下移
                return putRight(father.right, value);
            }
        } else {
            putLeft(index, value);
        }
        return true;
    }

    private boolean putLeft(int index, int value) {
        Node father = arr[index];
        if (value <= father.value) {
            if (father.left == 0) {// 右节点为空，直接放置
                father.left = getBouble();
                arr[father.left].has = true;
                arr[father.left].value = value;
                arr[father.left].father = index;
                if (father.right == 0 && (arr[father.father].left==0)){
                    zZip(father.father);
                } else if (arr[father.father].left == 0 && father.left == 0 && last > 2) {
                    rorateLeft(father.father, 2);

                }
            } else {// 该子树满，左链下移
                return putLeft(father.left, value);
            }
        } else {
            putRight(index, value);
        }
        return true;
    }

    private void cZip(int index) {
        arr[index].right = arr[arr[index].left].right;
        arr[arr[index].right].father = index;
        arr[arr[index].left].right = 0;

    }

    private void zZip(int index) {
        arr[index].left = arr[arr[index].right].left;
        arr[arr[index].left].father = index;
        arr[arr[index].right].left = 0;

    }

    public boolean append(int value) {
        lock.lock();
        Node father = new Node();
        if (arr.length == last) {
            expand();
        } else if (arr[0] == null) {
            for (int i = 0; i < arr.length; i++) {
                arr[i] = new Node();
            }
        }
        int index = search(value, 0);

        father = arr[index];
        if (!father.has) {
            father.value = value;
            father.has = true;
        } else {
            if (father.value >= value) {
                putLeft(index, value);
            } else {
                putRight(index, value);
            }

        }
        last++;
        lock.unlock();
        return true;

    }

    public boolean remove(int index) {
        lock.lock();
        if (arr[index].left != 0) {
            int tempIndex = arr[index].left;
            arr[index] = arr[arr[index].left];
            arr[arr[index].left].father = index;
            arr[arr[index].right].father = index;
            arr[tempIndex] = new Node();

        } else if (arr[index].right != 0) {
            int tempIndex = arr[index].right;
            arr[index] = arr[arr[index].right];
            arr[arr[index].left].father = index;
            arr[arr[index].right].father = index;
            arr[tempIndex] = new Node();
        }
        last--;
        // balance();
        lock.unlock();
        return true;
    }

    public int maxIndex(int index) {
        for (;;) {
            if (arr[index].right != 0) {
                index = arr[index].right;
            } else {
                return index;
            }
        }
    }

    public int minIndex(int index) {
        for (;;) {
            if (arr[index].left != 0) {
                index = arr[index].left;
            } else {
                return index;
            }
        }
    }

    private boolean rorateRight(int index, int count) {

        if (count == 1) {

            arr[index].right = arr[index].left;
            arr[index].left = 0;
            int temp = arr[index].value;
            arr[index].value = arr[arr[index].right].value;
            arr[arr[index].right].value = temp;
            return true;
        } else if (count == 2) {

            arr[index].right = arr[index].left;// arr[arr[index].right].right;
            arr[index].left = arr[arr[index].left].left;
            arr[arr[index].right].left = 0;
            arr[arr[index].left].father = index;

            int temp = arr[index].value;
            arr[index].value = arr[arr[index].right].value;
            arr[arr[index].right].value = temp;
            return true;
        } else {
            Node maxLeft = arr[maxIndex(index)];
            int temp = arr[index].value;
            arr[maxLeft.father].right = maxLeft.left;
            arr[maxLeft.left].father = maxLeft.father;
            arr[index].value = maxLeft.value;
            maxLeft.has = false;
            maxLeft.left = 0;
            maxLeft.right = 0;
            maxLeft.value = 0;
            return putRight(index, temp);
        }
    }

    private boolean rorateLeft(int index, int count) {

        if (count == 1) {

            arr[index].left = arr[index].right;
            arr[index].right = 0;
            int temp = arr[index].value;
            arr[index].value = arr[arr[index].left].value;
            arr[arr[index].left].value = temp;
            return true;
        } else if (count == 2) {

            arr[index].left = arr[index].right;// arr[arr[index].right].right;
            arr[index].right = arr[arr[index].right].right;
            arr[arr[index].left].right = 0;
            arr[arr[index].right].father = index;
            int temp = arr[index].value;
            arr[index].value = arr[arr[index].left].value;
            arr[arr[index].left].value = temp;
            return true;
        } else {

            Node minRight = arr[minIndex(index)];
            int temp = arr[index].value;
            arr[minRight.father].left = minRight.left;
            arr[minRight.left].father = minRight.father;
            arr[index].value = minRight.value;

            minRight.has = false;
            minRight.left = 0;
            minRight.right = 0;
            minRight.value = 0;
            return putLeft(index, temp);
        }
    }

    private boolean balance(int index) {
        int stepLeft = 0;
        int left = index;
        for (;;) {
            if (arr[left].left != 0) {
                stepLeft++;
                left = arr[left].left;
            } else if (arr[left].right != 0 && left != index) {
                stepLeft++;
                break;
            } else {
                break;
            }

        }
        int stepRight = 0;
        int right = index;
        for (;;) {
            if (arr[right].right != 0) {
                stepRight++;
                right = arr[right].right;
            } else if (arr[right].left != 0 && right != index) {
                stepRight++;
                break;
            } else {
                break;
            }
        }
        if (stepLeft - stepRight > 2) {
            rorateRight(index, 3);
        }
        if (stepRight - stepLeft > 2) {
            rorateLeft(index, 3);
        }
        return true;
    }

    @Override
    public String toString() {
        StringBuilder s = new StringBuilder();
        for (Node var : arr) {
            s.append(var.value + " ,");
        }
        return s.toString();
    }

}
```