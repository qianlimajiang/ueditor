# 说明 
此扩展时将Ueditor富文本编辑器的PHP上传代码重新封装，可支持阿里云/七牛云上传，默认配置文件在当前扩展下的src目录中，名为Config.php。
配置可以自定义，例如tp5框架，将本配置文件内的$config数组变量复制到application/目录，记得要先新建配置文件，不要直接覆盖。
默认配置未开启云上传，需将配置文件中的oss设置为true。如果使用了云端上传功能，需要下载云上传扩展包，比如阿里云oss的aliyuncs/oss-sdk-php。

# 实例代码（tp5为例）
<?php

namespace app\admin\controller;

use OSS\OssClient;
use OSS\Core\OssException;
use think\Exception;
use UeditorExtend\Ueditor as UeditorExtendUeditor;

class Ueditor extends Base
{
    public function index()
    {
        $get = $this->request->get();

        $config = include(APP_PATH . '/admin/extra/ueditor.php');

        $ueditor = new UeditorExtendUeditor($get, $config);

        //如果没有开启云上传，则不用写回调函数，参数为空就好
        $res = $ueditor->upload(function ($object, $file) {
            $accessKeyId = "阿里云oss-key";
            $accessKeySecret = "阿里云密钥";
            $endpoint = "oss-cn-beijing.aliyuncs.com";

            try {
                $ossClient = new OssClient($accessKeyId, $accessKeySecret, $endpoint);
            } catch (OssException $e) {
                throw new Exception($e->getMessage());
            }

            $bucket = "test";

            try {
                $result = $ossClient->putObject($bucket, $object, $file);
            } catch (OssException $e) {
                throw new Exception($e->getMessage());
            }
            return $result['info']['url'];
        });

        return json($res);
    }
}
