# 说明 
此扩展时将Ueditor富文本编辑器的PHP上传代码重新封装，可支持阿里云/七牛云上传，默认配置文件在当前扩展下的src目录中，名为Config.php。
配置可以自定义，例如tp5框架，将本配置文件内的$config数组变量复制到application/目录，记得要先新建配置文件，不要直接覆盖。
默认配置未开启云上传，需将配置文件中的oss设置为true。如果使用了云端上传功能，需要下载云上传扩展包，比如阿里云oss的aliyuncs/oss-sdk-php。

# 实例代码（tp5为例）
namespace app\admin\controller;

use OSS\OssClient;
use OSS\Core\OssException;
use UeditorExtend\Ueditor as UeditorExtendUeditor;

class Ueditor extends Base
{
    public function index()
    {
        $get = $this->request->get();

        $config = include(APP_PATH . '/admin/extra/ueditor.php');
        //如果开启了oss则在这里增加判断即可，如果没有开启，可忽略此处代码
        if ($config['oss'] && $_FILES && isset($_FILES['upfile']['tmp_name'])) {
            $accessKeyId = "**************";
            $accessKeySecret = "**************";
            $endpoint = "oss-cn-beijing.aliyuncs.com";

            try {
                $ossClient = new OssClient($accessKeyId, $accessKeySecret, $endpoint);
            } catch (OssException $e) {
                print $e->getMessage();
            }

            $bucket = "******";
            $object = uniqid() . '.jpg'; //文件名称
            try {
                $result = $ossClient->uploadFile($bucket, $object, $_FILES['upfile']['tmp_name']);
            } catch (OssException $e) {
                print $e->getMessage();
            }
            $res_url = $result['info']['url'];
            
            $get['action'] = 'catchimage';
            $fieldName = $config['catcherFieldName'];
            $_POST[$fieldName] = [$res_url];
        }
        //如果没有开启oss上传，直接写这两行代码即可
        $ueditor = new UeditorExtendUeditor($get, $config);
        $res = $ueditor->upload();

        return json($res);
    }
}
