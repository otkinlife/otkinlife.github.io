---
layout: post
title: 设计模式：工厂模式
date: 2021-02-17
tags: ["设计模式","设计模式"]
categories: 设计模式

---

<!-- wp:paragraph -->

今天总结一下简单工厂模式，工厂模式、抽象工厂模式

<!-- /wp:paragraph -->

<!-- wp:heading {"level":4} -->

#### 简单工厂模式

<!-- /wp:heading -->

<!-- wp:paragraph -->

假设我们现在要代理一家餐店，这个餐店主要卖鸡肉产品，提供了几种烹饪方式，包括油炸、烧烤、煲汤。我们希望顾客点单的时候告诉我们烹饪方式就能快速出餐。

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

我们发现这几种方式都是对鸡肉产品的处理，有一定的通性，可以进行一定程度的抽象，我们使用简单工厂来实现

<!-- /wp:paragraph -->

<!-- wp:code -->

    class ProductChiken {
        public getProduct(type) {
            swtich (type) {
                case "fry":
                    return  new fry_chiken()
                case "barbecue":
                    return new barbecue_chicken()
                case "soup":   
                    return new soup_chiken()   
            }
        }
    }

    class barbecue_chicken {}
    class fry_chicken {}
    class soup_chicken {}

<!-- /wp:code -->

<!-- wp:paragraph -->

以上就是简单工厂的伪代码了，但是如果我想新增一个烹饪方式的话就需要调整swich里的选项，每次调整都要改动原代码不是一个好的选择，我们进一步使用工厂模式来优化

<!-- /wp:paragraph -->

<!-- wp:code -->

    interface Product {
        public getProduct()
    }

    class ProductBarbecueChiken: Product {
        public getProduct() {
            return new barbecue_chicken()      
        }
    }

    class ProductFryChiken: Product {
        public getProduct() {
            return new fry_chicken()      
        }
    }

    class ProductSoupeChiken: Product {
        public getProduct() {
            return new soup_chicken()      
        }
    }

    class barbecue_chicken {}
    class fry_chicken {}
    class soup_chicken {}

<!-- /wp:code -->

<!-- wp:paragraph -->

工厂模式将原来简单工厂的判断逻辑抽离出来，在客户端侧直接实例化需要的产品对象即可，这样新增烹饪方式也不需要修改原来的代码了。看起来问题已经都解决了，可是因为生意太好了，现在需要考虑业务扩展。除了鸡肉产品我们还计划做一些猪肉的。这个时候就可以用到抽象工厂模式了

<!-- /wp:paragraph -->

<!-- wp:code -->

    //定义工厂接口类
    interface Factory {
        public Barbecue()
        public Fry()
        public Soup()
    }

    //实现鸡肉产品工厂
    class FactoryChicken: Factory {
        public Barbecue() {
            return new barbecue_chicken() 
        }
        public Fry() {
            return new fry_chicken() 
        }
        public Soup() {
            return new soup_chicken()
        }
    }

    //实现猪肉工厂产品
    class FactoryMeet: Factory {
        public Barbecue() {
            return new barbecue_meet() 
        }
        public Fry() {
            return new fry_meet() 
        }
        public Soup() {
            return new soup_meet()
        }
    }

    //具体产品
    class barbecue_chicken {}
    class fry_chicken {}
    class soup_chicken {}

    class barbecue_meet {}
    class fry_meet {}
    class soup_meet {}

<!-- /wp:code -->