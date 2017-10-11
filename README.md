#  think php 5的简单应用

* [简介](#简介)
* [登录](#登录)

## 简介
框架的起始文件在application/index/controller下的Index.php文件
application/index/model下文件与数据库表名对应

## 用户相关操作
可以创建个基类，保存token验证的方法和一些常用的输出
```
class BaseController extends Controller
{
    /**
     * 验证token
     */
    protected function checkToken(){
        $username = request() -> post('username');
        $token = request() -> post('token');
        if(!$username || !$token){
            $this->echoToken();
            return;
        }
        $user = User::where([
            'username' => $username,
            'token' => $token
        ])->find();
        if(!$user){
            $this->echoToken();
            return;
        }
        $time1 = strtotime(date('Y-m-d H:i:s'));
        $time2 = strtotime($user->update_time);
        $disTime = ($time1 - $time2);
        if($disTime > 60 * 30){
            $this->echoToken();
            return;
        }else{
            return $user;
        }
    }
    function echoParams(){
        echo json_encode(array(
            'code' => 209,
            'msg' => '参数错误'
        ));
    }
    function echoToken(){
        echo json_encode(array(
            'code' => 210,
            'msg' => 'token异常'
        ));
    }
}
```
登录
```
    /**
     * 登录
     */
    public function login(){
        $username = \request()->post('username');
        $password = \request()->post('password');
        if(!$username || !$password){
            $this->echoParams();
            return ;
        }
        $res = User::where([
            'username' => $username,
            'password' => $password
        ])->find();
        if($res){
            $token = \request() -> token('__token__', 'sha1');
            $res -> token = $token;
            $res = $res -> save();
            if($res){
                echo json_encode(array(
                    'code'  => 200,
                    'msg' => '登录成功',
                    'data' => [
                        'token' => $token
                    ]
                ));
            }else{
                echo json_encode(array(
                    'code' => 201,
                    'msg' => '登录失败'
                ));
            }
            return;
        }else{
            echo json_encode(array(
                'code' => 202,
                'msg' => '用户名或者密码错误'
            ));
        }
    }
```
token登录
```
    /**
     * token登录
     */
    public function loginToken(){
        $res = $this->checkToken();
        if($res){
            $res->update_time = time();
            $res = $res->save();
            if($res){
                echo json_encode(array(
                    'code' => 200,
                    'msg' => '登录成功'
                ));
            }else{
                echo json_encode(array(
                    'code' => 201,
                    'msg' => '登录失败'
                ));
            }
        }
    }
```
注册
```
    /**
     * 注册
     */
    public function register(){
        $username = \request()->post('username');
        $password = \request()->post('password');
        if(!$username || !$password){
            echo json_encode(array(
                'code' => 209,
                'msg' => '参数错误'
            ));
            return ;
        }
        $res = User::where('username', $username)->find();
        if($res){
            echo json_encode(array(
                'code' => 202,
                'msg' => '用户名已存在'
            ));
            return ;
        }
        $res = User::create([
            'username' => $username,
            'password' => $password
        ]);
        if($res){
            echo json_encode(array(
                'code' => 200,
                'msg' => '账户创建成功',
                'data' => [
                    'username' => $res -> username
                ]
            ));
        }else{
            echo json_encode(array(
                'code' => 201,
                'msg' => '账户创建失败'
            ));
        }
    }
```
登出
```
    /**
     * 登出
     */
    public function logout(){
        $username = \request()->post('username');
        if(!$username){
            $this->echoParams();
            return;
        }
        $res = User::where('username', $username) -> find();
        if($res){
            $res -> token = "";
            $res -> save();
            echo json_encode(array(
                'code' => 200,
                'msg' => '登出成功'
            ));
        }else{
            echo json_encode(array(
                'code' => 201,
                'msg' => '登出失败'
            ));
        }
    }
```
头像上传
```
    /**
     * 头像上传
     */
    public function uploadPhoto(){
        $res = $this->checkToken();
        if(!$res)
            return;
        // 获取表单上传文件 例如上传了001.jpg
        $file = request()->file('file');
        if($file == null){
            $this->echoParams();
            return;
        }
        // 移动到框架应用根目录/public/uploads/ 目录下
        $fileName = $_SERVER['DOCUMENT_ROOT']."/php6/update/picture/";
        if(!file_exists($fileName)){
            mkdir($fileName, 0777, true);
        }
        $info = $file->move($fileName);
        if($info){
            // 成功上传后 获取上传信息
            // 输出 jpg
//                echo $info->getExtension();
            // 输出 20160820/42a79759f284b767dfcb2a0197904287.jpg
            $savePath = $info->getSaveName();
            // 输出 42a79759f284b767dfcb2a0197904287.jpg
//                echo $info->getFilename();
            $res = Information::create([
                'user_id' => $res->user_id,
                'photo' => $savePath
            ]);
            if($res){
                echo json_encode(array(
                    "code" => 200,
                    "msg" => "上传成功",
                    "save_path" => $savePath
                ));
            }else{
                echo json_encode(array(
                    "code" => 201,
                    "msg" => "上传失败"
                ));
            }

        }else{
            // 上传失败获取错误信息
//                echo $file->getError();
            echo json_encode(array(
                "code" => 202,
                "msg" => "上传失败"
            ));
        }
    }
```


