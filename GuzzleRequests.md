# PHP Class To Use Guzzle More Easier For Sending Requests
```php
<?php 
use GuzzleHttp\Client;
use GuzzleHttp\Cookie\CookieJar;
class SendGuzzleRequest extends Controller
{
    protected $request;
    protected $cookies;
    public function __construct(){}

    public function send($data){
        $defaultData = [
            'url' => 'http://example.com',
            'method' => 'GET',
            'data' => [],
            'headers' => [],
            'cookies' => [],
            'redirect' => [],
        ];
        $data = array_merge($defaultData, $data);

        $cookies = !empty($data['cookies']);

        if($cookies){
            $client = new Client(['cookies' => true]);
            if(is_object($data['cookies'])){
                $cookieJar = $data['cookies'];
            }else{
                $cookieJar = new CookieJar();
            }
        }else{
            $client = new Client();
        }

        $request = [];
        $request[0] = $data['method'];
        $request[1] = $data['url'];
        $request[2] = [];
        $request[2]['headers'] = $data['headers'];

        if($cookies && is_array($data['cookies'])){
            $cookieJar = CookieJar::fromArray(...$data['cookies']);
        }

        if($data['method'] == 'GET'){
            $request[2]['query'] = $data['data'];
            $this->request = $client->request(...$request);
        }elseif($data['method'] == 'POST'){
            $request[2]['form_params'] = $data['data'];
            $request[2]['cookies'] = $cookieJar;
            $this->request = $client->request(...$request);
        }

        if(isset($cookieJar)){
            $this->cookies = $cookieJar;
        }
    }

    public function status(){
        return $this->request->getStatusCode();
    }

    public function response(){
        return $this->request->getBody()->getContents();
    }

    public function getCookies(){
        return $this->cookies;
    }
}

```
