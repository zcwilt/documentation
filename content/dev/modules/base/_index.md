---
title: Base Module Support
description: 
weight: 10
layout: docs
---

{{% alert title="Warning" color="warning" %}}
The documentation in this section is Work in progress and relates to features that might not be available yet.
{{% /alert %}}

# Introduction

The base module support classes contain common code that is used across all module types e.g Payments, Shipping and Order Total modules.

It also provides some common helper methods that you can use in you modules.

## Accessing Define Values

All module types store information about themselves in the database `configuration` table.

The `keys` for this table will look something like 

- MODULE_PAYMENT_COD_STATUS
- MODULE_SHIPPING_FLAT_COST
- MODULE_ORDERTOTAL_TOTAL_SORT_ORDER

The structure of these defines is  

MODULE_[module type]\_[module name]\_[define name]

where module type is one of `PAYMENT` or `SHIPPING` or `ORDERTOTAL`

The module name is taken from the $code class variable of the module, and the define name describes the purpose of the define. 

In previous version of Zen Cart,these configuration values were accessed directly using the key that was stored in the database, however using 
the new module support classes we access them through a helper method.

e.g In legacy code we might have done 

`$this->sort_order = defined('MODULE_PAYMENT_COD_SORT_ORDER') ? MODULE_PAYMENT_COD_SORT_ORDER : null;`

With the new infratructure we can simply do 

`$this->sort_order = $this->getDefine('SORT_ORDER);`

The `getDefine` method is context aware, so knows which type of module you are using(PAYMENT/SHIPPING/ORDERTOTAL) and the current define name (e.g. COD) for 
the module you are using.

Furthermore, you can also set a value that will be returned from the method if the configuration value does not exist. 

In the example above, the method will return `null` if the `MODULE_PAYMENT_COD_SORT_ORDER` does not exist. 

However you could also do `$this->sort_order = $this->getDefine('SORT_ORDER, 0);`

and the method will return `0`, if the configuration value does not exist.

There are also a number of helper methods to get some of the most common module configuration keys. 

### getTitle

### getAdminTitle

### getDescription

### getSortOrder

### getZone





