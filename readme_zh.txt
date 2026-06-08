开始配置客户端之前，你需要了解以下内容：
 1.) 你的服务器 principal URL
 2.) 跨域和非跨域配置的区别
 3.) 跨域配置的问题及解决方法
 4.) Digest 认证的问题及解决方法（如果你的服务器使用 Digest 认证）
 5.) SSL/https 和无效（或自签名）证书的问题
 6.) 选择合适的配置类型（支持 3 种配置类型）
 7.) HTML5 缓存更新
 8.) 通用安装说明
 9.) DAViCal（非跨域）安装说明

1.) 你的 principal URL
    - 什么是 principal URL？
      请查看你的服务器文档！
      principal URL 示例（<USERNAME> = 你的用户名）：
        http://davical.server.com/caldav.php/<USERNAME>/ （DAViCal 示例）
        http://baikal.server.com/card.php/principals/<USERNAME>/ （Baïkal 示例）
        http://radicale.server.com:5232/<USERNAME>/ （Radicale 示例）
        http://osx.server.com:8008/principals/users/<USERNAME>/ （OS X 示例 1）
        https://osx.server.com:8443/principals/users/<USERNAME>/ （OS X 示例 2）

2.) 跨域 / 非跨域配置
    - 什么是跨域配置？
      如果服务器 origin 与客户端安装的 origin 不同，则你的配置就是跨域的！
    - 什么是 origin？
      origin 是不包含"完整路径"的 URL => <协议>://<域名>:<端口>
      示例 1：
        URL:    http://davical.server.com/caldav.php/<USERNAME>/
        origin: http://davical.server.com:80 （http 默认端口为 80）
      示例 2：
        URL:    https://davical.server.com/caldav.php/<USERNAME>/
        origin: https://davical.server.com:443 （https 默认端口为 443）
      示例 3：
        URL:    http://lion.server.com:8008/principals/users/<USERNAME>/
        origin: http://lion.server.com:8008
    - 我的服务器 origin 是什么？
      就是你的 principal URL 的 origin
    - 完整示例？
      示例 1：
        principal URL: https://lion.server.com:8443/principals/users/<USERNAME>/ （服务器 URL）
        客户端 URL:    https://www.server.com/client/ （客户端安装 URL）
        =>
        服务器 origin: https://lion.server.com:8443
        客户端 origin: https://www.server.com:443
        这是跨域配置吗？是（服务器 origin != 客户端 origin）
      示例 2：
        principal URL: http://davical.server.com/caldav.php/<USERNAME>/ （服务器 URL）
        客户端 URL:    http://davical.server.com/client/ （客户端安装 URL）
        =>
        服务器 origin: http://davical.server.com:80
        客户端 origin: http://davical.server.com:80
        这是跨域配置吗？否（服务器 origin == 客户端 origin）
    注意：如果检测到跨域配置，浏览器控制台会显示警告！
    注意：跨域配置会自动检测，无需在 config.js 中手动设置！

3.) 跨域配置的问题及解决方法（如果你使用跨域配置）
    - 为什么跨域配置会有问题？
      客户端使用 JavaScript 编写，而 JavaScript 有一个主要的安全限制（内置于浏览器中）：
        如果你使用跨域配置，但服务器没有返回正确的 HTTP CORS 头（参见 http://www.w3.org/TR/cors/），
        则 JavaScript 会拒绝向服务器发送请求（更准确地说：浏览器会先发送一个 OPTIONS 请求（称为预检请求）
        来检查服务器返回的 HTTP 头，如果没有返回正确的 CORS 头，则实际请求不会发送！）。
    - 如何解决这个问题？
      a.) 你的服务器必须返回以下额外的 HTTP 头：
            Access-Control-Allow-Origin: *
            Access-Control-Allow-Methods: GET, POST, OPTIONS, PROPFIND, PROPPATCH, REPORT, PUT, MOVE, DELETE, LOCK, UNLOCK
            Access-Control-Allow-Headers: User-Agent, Authorization, Content-type, Depth, If-match, If-None-Match, Lock-Token, Timeout, Destination, Overwrite, Prefer, X-client, X-Requested-With
            Access-Control-Expose-Headers: Etag, Preference-Applied
      b.) 如果浏览器发送了 Access-Control-Request-Method 头（CORS 定义的预检请求），服务器必须在不需要认证的情况下为 OPTIONS 请求返回这些头，并且必须返回 200（或 2xx）HTTP 状态码（成功）。
    - 如何在 CardDAV/CalDAV 服务器上添加这些头？
      查看服务器文档，或联系服务器开发者，询问 CORS 或自定义 HTTP 头的支持。
    - 如果服务器不支持 CORS 或自定义 HTTP 头，如何添加？
      在 Web 服务器（或代理服务器）配置中添加自定义头（如果可能）—— 参见 misc/config_davical.txt 中的 Apache 示例。

4.) Digest 认证的问题及解决方法（如果你的服务器使用 Digest 认证）
    - 为什么 Digest 认证会有问题？
      很多浏览器对 Digest 认证的支持存在错误或缺陷（尤其是通过 JavaScript 使用时）。
    - 如何解决？
      a.) 在服务器配置中禁用 Digest 认证，启用 Basic 认证（注意：使用 Basic 认证时必须使用 SSL/https！）
      b.) 替代方案（如果无法切换到 Basic 认证）：可以尝试在 config.js 中启用 globalUseJqueryAuth 选项（注意：不保证在所有浏览器中都能正常工作）
      注意：如果要使用 auth 模块（参见下方 6.) c.)），必须使用 Basic 认证（该模块不支持 Digest 认证）！

5.) SSL/https 和无效（或自签名）证书的问题
    - 为什么客户端无法连接到具有无效/自签名证书的服务器？
      如果用户打开网页时浏览器检测到无效/自签名证书，浏览器会警告用户，通常还会提供一个手动接受服务器证书（或添加安全例外）的选项。但是，如果是 JavaScript 发送的请求，则无法向用户显示安全警告，也无法通过 JavaScript 直接添加安全例外！
    - 如何解决？
      a.) 使用来自商业 CA 的有效服务器证书，或
      b.) 如果服务器证书不是自签名的，而是由你自己的 CA 签发的，将你的 CA 证书添加到浏览器/系统的"受信任的根证书"中，或
      c.) 在浏览器中直接打开 principal URL，手动接受无效证书（或添加安全例外）

6.) 客户端配置类型
    - 客户端支持哪些配置类型？
      a.) 静态配置：在 config.js 中预定义 principal URL、用户名和密码。使用 globalAccountSettings（而不是 globalNetworkCheckSettings 或 globalNetworkAccountSettings），并在 config.js 中将 href 选项设置为完整的 principal URL。
          - 优点：登录过程快速，无需输入用户名/密码（没有登录界面）
          - 缺点：用户名/密码在 config.js 中可见（仅推荐用于内网或家庭环境）
      b.) 标准配置：显示登录界面，需要用户输入有效的用户名和密码。使用 globalNetworkCheckSettings（而不是 globalAccountSettings 或 globalNetworkAccountSettings），并在 config.js 中将 href 选项设置为不含用户名部分的 principal URL（用户名会从登录界面追加到 href 值之后）。
          - 优点：用户名/密码由用户输入（config.js 中不保存可见的用户名/密码）
          - 缺点：如果用户输入了错误的用户名/密码，浏览器会弹出认证窗口（JavaScript 无法禁用此行为；见下一种配置）
      c.) 特殊配置：将用户名/密码发送到 PHP auth 模块（auth 目录），该模块会验证用户名/密码是否正确，认证成功后返回配置 XML（需要额外配置；返回的 XML 与 globalAccountSettings /a.)/ 配置选项处理方式完全相同）。使用 globalNetworkAccountSettings（而不是 globalAccountSettings 或 globalNetworkCheckSettings），并将 href 设置为 auth 目录的 URL（如果 auth 目录位于客户端安装子目录中，则使用默认值）。在 b.) 配置可以正常工作后，再使用此配置来解决认证弹窗问题。
          - 优点：输入错误的用户名/密码不会弹出认证窗口，支持动态 XML 配置生成（可以通过修改模块配置或 PHP 代码为不同用户生成不同的配置）
          - 缺点：需要 PHP >= 5.3 和额外配置，仅支持 Basic 认证
          Auth 模块配置：
              - 更新 auth/config.inc：
                  将 $config['auth_method'] 设置为 'generic'（默认值）
                  设置 $config['accounts'] —— 通常只需要修改 href 值中的 "http://www.server.com:80" 部分，但也可以修改 syncinterval 和 timeout 值
                  将 $config['auth_send_authenticate_header'] 设置为 true
              - 更新 auth/plugins/generic_conf.inc：
                  将 $pluginconfig['base_url'] 设置为服务器 origin
                  将 $pluginconfig['request'] 设置为服务器路径（例如 DAViCal: '/caldav.php'）
              - 在浏览器中手动访问 auth 目录，输入用户名和密码 —— 你将获得适用于你的安装的配置 XML（如果没有，请重新检查之前的设置！）
                  注意：返回的 XML 内容与 globalAccountSettings /a.)/ 配置选项处理方式相同
              - 更新 auth/config.inc：
                  将 $config['auth_send_authenticate_header'] 设置回 false

7.) HTML5 缓存更新
    每次更新配置或任何其他文件后，必须执行 cache_update.sh 脚本（否则浏览器将继续使用存储在 HTML5 缓存中的旧版本文件）；或者手动更新 cache.manifest —— 将以 "#V 20" 开头的第二行修改为其他内容（该文件只需要"某些"更改）

8.) 通用安装说明
    a.) 先阅读上述 1-7 部分 :-)
    b.) 将源代码复制到你的 Web 服务器目录中（如果使用 Apache，强烈建议启用以下模块：mod_expires、mod_mime 和 mod_deflate …… 详见 .htaccess）
    c.) 在 config.js 中设置 CalDAV 服务器相关配置（参见 6.)）
    d.) 在 config.js 中设置其他配置选项（参见 config.js 中的注释）
    e.) 更新 HTML5 缓存（参见 7.)）
    f.) 在浏览器中打开安装目录
    g.) 登录并使用客户端 :-)

9.) DAViCal（非跨域）安装说明
    a.) 将源代码复制到 DAViCal 的 "htdocs" 目录中（或复制到其他目录，并在 DAViCal 虚拟服务器配置中使用 Web 服务器别名，例如："Alias /client/ /usr/share/client/"）
    b.) 在浏览器中打开安装目录
    c.) 登录并使用客户端 :-) …… 注意：如果在 config.js 中修改了内容（非必需），请参见 7.)


如果出现问题，请检查浏览器控制台日志！
