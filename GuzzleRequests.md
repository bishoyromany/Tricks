# PHP Class To Use Guzzle More Easier For Sending Requests
```php
<?php
use GuzzleHttp\Client;
use GuzzleHttp\Cookie\CookieJar;
use GuzzleHttp\Exception\ClientException;
class SendGuzzleRequest extends Controller
{
    protected $request;
    protected $status;
    protected $response;
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
            'handleExceptions' => true,
        ];
        $data = array_merge($defaultData, $data);

        $cookies = !empty($data['cookies']);

        if($cookies){
            $client = new Client(['cookies' => true]);
            if(is_object($data['cookies'])){
                $cookieJar = $data['cookies'];
            }else{
                $cookieJar = new CookieJar();
                if($cookies && is_array($data['cookies'])){
                    $cookieJar = CookieJar::fromArray(...$data['cookies']);
                }
            }
        }else{
            $client = new Client();
        }

        $request = [];
        $request[0] = $data['method'];
        $request[1] = $data['url'];
        $request[2] = [];
        $request[2]['headers'] = $data['headers'];

        if($data['method'] == 'GET'){
            $request[2]['query'] = $data['data'];
        }elseif($data['method'] == 'POST'){
            $request[2]['form_params'] = $data['data'];
            if(isset($cookieJar)){
                $request[2]['cookies'] = $cookieJar;
            }
        }

        if($data['handleExceptions']){
            try{
                $this->request = $client->request(...$request);
            }catch(ClientException $e){
                $e = $e->getResponse();
                $this->status = $e->getStatusCode();
                $this->response = $e->getBody()->getContents();
                return $this;
            }
        }else{
            $this->request = $client->request(...$request);
        }
        $this->status = $this->request->getStatusCode();
        $this->response = $this->request->getBody()->getContents();
        if(isset($cookieJar)){
            $this->cookies = $cookieJar;
        }

        return $this;
    }

    public function getStatus(){
        return $this->status;
    }

    public function getResponse(){
        return $this->response;
    }

    public function getRequest(){
        return $this->request;
    }

    public function getCookies(){
        return $this->cookies;
    }
}

}

```
## Sample Usage
```php
require_once __DIR__.'/SendGuzzleRequest.php';
use SendGuzzleRequest as Http;

<?php 
    $requ = new Http();
    $requ->send([  
        'method' => 'GET',
        'url' => 'url here',
        'data' => [
            'data' => 'like any php array'
        ],
        'headers'   => [  
            'API-Key'      => 'auth key here for example etc..', 
            'Content-Type' => 'application/json'    
        ]
    ]);
    $status = $requ->getStatus();
    $data = json_decode($requ->getResponse());
    $cookies = $requ->getCookies();
    if($status != 200){
        return 'Request Response Is Not 200!!';
    }
```
