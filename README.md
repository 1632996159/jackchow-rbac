# Rbac (ThinkPHP 5 Package)

Rbac是向ThinkPHP 5添加基于角色的权限的简洁而灵活的方式。


## Contents

- [安装](#installation)
- [配置](#configuration)
    - [User relation to roles](#user-relation-to-roles)
    - [Models](#models)
        - [Role](#role)
        - [Permission](#permission)
        - [Admin](#admin)
- [使用](#usage)
    - [Concepts](#concepts)
        - [Checking for Roles & Permissions](#checking-for-roles--permissions)
        
- [故障排除](#troubleshooting)
- [License](#license)
- [贡献指南](#contribution-guidelines)
- [附加信息](#additional-information)

## 安装

为了安装ThinkPHP 5 Rbac，只需在composer.josn文件中添加

    "jackchow/rbac": "^1.0"

 然后运行 `composer install` 或者 `composer update`.


## 配置

在 `config/rbac.php` 文件中设置属性值.
委托使用这些值来引用正确的用户表和模型.

您还可以发布此包的配置以进一步自定义表名称和模型名称空间。
只需在application文件中添加下面数组代码:
```bash
return [
    'Jackchow\Rbac\Command\PublishCommand',
    'Jackchow\Rbac\Command\MigrateCommand',
];
```
使用`php think rbac：publish`和`rbac.php`文件就可以在你的config目录下创建。

### 用户与角色的关系

现在在上面注册了Rbac的生成命令后生成Rbac的迁移文件:

```bash
php think rbac:migrate
```

它将生成`<timestamp>_rbac.php` 迁移文件.
您现在可以使用artisan migrate命令运行它：
如果你的thinkphp5还没有安装migrate扩展包,请前往安装使用。[点击了解](https://www.kancloud.cn/manual/thinkphp5_1/354133)

```bash
php think migrate:run
```

迁移后，将出现五个新表格:
- `admins` &mdash; 保存用户表记录 
- `roles` &mdash; 保存角色表记录
- `permissions` &mdash; 保存权限表记录
- `role_user` &mdash; 保存用户和角色之间的 [多对多](https://www.kancloud.cn/manual/thinkphp5_1/354060) 关系 
- `permission_role` &mdash; 保存权限和角色之间的 [多对多](https://www.kancloud.cn/manual/thinkphp5_1/354060) 关系 

如果系统已经存在admins表 可以在生成的迁移文件注释相应的迁移代码。

生成迁移文件位置在：\database\migrations  中

hhh,相信你已经懂了.

### 模型

#### Role

使用以下示例在`application\admin\model\Roles.php`内创建角色模型：

```php
namespace app\admin\model;

use Jackchow\Rbac\RbacRole;

class Roles extends RbacRole
{

}
```

角色模型有三个主要属性:
- `name` &mdash; 角色的唯一名称，用于在应用程序层查找角色信息。 例如：“管理员”，“所有者”，“员工”.
- `description` &mdash; 该角色的人类可读名称。 不一定是唯一的和可选的。 例如：“用户管理员”，“项目所有者”，“Widget Co.员工”.

 `description` 字段是可选的; 他的字段在数据库中是可空的。

#### Permission

使用以下示例在`application\admin\model\Permissions.php`内创建角色模型：

```php
<?php

namespace app\admin\model;

use Jackchow\Rbac\RbacPermission;

class Permissions extends RbacPermission
{
}
```

“权限”模型与“角色”具有相同的两个属性：
- `name` &mdash; 权限的唯一名称，用于在应用程序层查找权限信息。一般保存的是路由.
- `description` &mdash; 该权限的描述。 例如：“创建文章”，“查看文件”，“文件管理”等权限.

 `description` 字段是可选的; 他的字段在数据库中是可空的。


#### Admin

接下来，在现有的Admin模型中使用`RbacUserTrait`特征。 例如：

```php
<?php

namespace app\admin\model;

use think\Model;
use Jackchow\Rbac\Traits\RbacUser;

class Admins extends Model
{
    use RbacUser;

    protected $hidden=['password','created_at','updated_at'];

}
```

这将启用与Role的关系，并在User模型中添加以下方法roles（）和 can（$ permission）


不要忘记转储composer的自动加载

```bash
composer dump-autoload
```

**你准备好了.**

## 使用

### 概念
让我们从创建以下`角色`和'权限'开始：

```php
$owner = new Role();
$owner->name         = 'owner';
$owner->desription = '网站所有者'; // 可选
$owner->save();

$admin = new Role();
$admin->name         = 'admin';
$admin->desription = '管理员'; // 可选
$admin->save();
```

接下来，创建两个角色让我们将它们分配给用户。
由于`HasRole`特性，这很容易：

```php
$admin = Admin::where('username', 'jack')->find();

// 为用户分配角色

$user->roles()->attach($admin->id); 
```

现在我们只需要为这些角色添加权限：

```php
$createPost = new Permission();
$createPost->name         = 'post/edit';
// 允许用户...
$createPost->description  = '编辑一篇文章'; // 可选
$createPost->save();

$editUser = new Permission();
$editUser->name         = 'post/create';
// 允许用户...
$editUser->description  = '创建一篇文章'; // optional
$editUser->save();


$admin->perms()->sync(array($createPost->id));


$owner->perms()->sync(array($createPost->id, $editUser->id));
```

#### 检查用户是否拥有权限

现在我们可以通过执行以下操作来检查角色和权限：

```php

$admin->can('post/edit');   // false
$admin->can('post/create'); // true
```

到目前为止，已经可以很大的满足到后台用户权限管理功能了。

本功能目前比较简单，现在作者正在扩展其他功能.

最后，如果你觉得不错，请start一个吧 你的鼓励是我创作的无限动力.

## 故障排查

如果你迁移和配置中遇到问题，可直接联企鹅号:775893055



## License

Rbac is free software distributed under the terms of the MIT license.

## Contribution guidelines

Support follows PSR-1 and PSR-4 PHP coding standards, and semantic versioning.

Please report any issue you find in the issues page.  
Pull requests are welcome.
