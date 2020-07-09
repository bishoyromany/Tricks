# This Class Contains Some Pretty Good Helper Functions
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class HelpersController extends Controller
{   
    /**
     * ! Class Map
     * -----------------------------------------------------------------------------------
     * ? test: Test Method                                                               |
     * -----------------------------------------------------------------------------------
     * ? modelManager: Model Fetch Data Manager                                          |
     * -----------------------------------------------------------------------------------
     * ? valid: Array Data Validator                                                     |
     * -----------------------------------------------------------------------------------
     * ? responseHandler: HTTP API Response Handler                                      |
     * -----------------------------------------------------------------------------------     
     * ? modelInsert: Model Records Creator                                              |
     * -----------------------------------------------------------------------------------
     * ? modelUpdate: Model Records Updater                                              |
     * -----------------------------------------------------------------------------------
     * ? modelShow: One Record Fetch                                                     |
     * -----------------------------------------------------------------------------------
     * ? modelDelete: Model Records Delete                                               |
     * -----------------------------------------------------------------------------------
     */


    /**
     * Test Method
     * ? This Method Will Return A Test Text
     * @return String
     */
    public static function test(): String {
        return "test is done";
    }

    /**
     * Model Fetch Data Manager
     * ? This Method Will Fetch Data We Need From Given Model In Easy & Dynamic And Easy Way
     * TODO: Allow Custom Search
     * @param params Array        | The Pagination, Order, Select And Search Data
     * @param model String        | Model Class Namesapce
     * @param searchList Array    | Search Columns List
     * @param query Boolean|Array | Custom Search Param, If Passed Should Be Array Else Boolean
     * @return Object
     */
    public static function modelManager(Array $params, String $model, Array $searchList, $query = false) : Object {
        /**
         * Define All Paramteres
         */
        $page = isset($params['page']) ? (int)$params['page'] : 0;
        $search = isset($params['search']) ? $params['search'] : '';
        $perpage = isset($params['rowsPerPage']) ? (int)$params['rowsPerPage'] : 10;
        $offset = $page == 0 ? 0 : $page * $perpage;
        $orderBy = isset($params['orderBy']) ? (string)$params['orderBy'] : 'id';
        $order = isset($params['order']) ? (string)$params['order'] : 'ASC';
        $select = isset($params['select']) ? (array)$params['select'] : ['*'];

        /**
         * Create An Instance From The Class Name
         */
        $model = app($model);

        /**
         * Start The Query
         */
        $fullQuery = $model->select($select);

        /**
         * Bind All Search Columns List
         */
        for($i = 0; $i < count($searchList); $i++){
            if($i == 0){
                $fullQuery = $fullQuery->where($searchList[$i] , 'LIKE', '%'.$search.'%');
                continue;
            }
            $fullQuery = $fullQuery->orWhere($searchList[$i] , 'LIKE', '%'.$search.'%');
        }   

        /**
         * Get Total Records Count, Without Using Options
         */
        $count = $fullQuery->count();

        /**
         * Get Found Data Using Options
         */
        $data = $fullQuery->orderBy($orderBy, $order)->offset($offset)->limit($perpage)->get();

        /**
         * Return The Result
         */
        return self::responseHandler([
            'data' => $data,
            'total' => $count,
            'page' => $page,
            'perpage' => $perpage,
            'offset' => $offset,
            'orderBy' => $orderBy,
            'order' => $order,
            'select' => $select,
        ], 'success');
    }

    /**
     * Array Data Validator
     * ? This Method Will Validate Imported Data
     * TODO: Allow Custom Error Response
     * @param data Array    | The Data You Wanna Validate
     * @param rules Array   | The Validate Rules
     * @return Object|Boolean | Object If Found Errors | Boolean If No Errors Found 
     */
    public static function valid($data, $rules){
        $validator = \Illuminate\Support\Facades\Validator::make($data, $rules);

        if($validator->fails()){
            return self::responseHandler($validator->errors(), 'error');
        }

        return false;
    }

    /**
     * HTTP API Response Handler
     * ? This Method take should handle all API responses
     * @param resp String|Array   | The Response Text
     * @param type String|Integer | The Response Status Code
     * @return Object
     */
    public static function responseHandler($resp, $type) : Object {
        $status = 400;
        $res = [];
        if(is_string($type)){
            if($type == 'not_found'){
                $status = 404;
            }elseif($type == 'error'){
                $status = 400;
            }elseif($type == 'success'){
                $status = 200;
            }elseif($type == 'insert'){
                $status = 201;
            }elseif($type == 'delete'){
                $status = 202;
            }elseif($type == 'update'){
                $status = 201;
            }
        }else{
            $status = $type;
        }

        if(!is_string($resp)){
            $res = $resp;
        }else{
            $res['message'] = $resp;
        }
        
        return response()->json($res, $status);
    }

    /**
     * Model Records Creator
     * ? This Method Will Create New Record From Given Params
     * TODO: Allow Inserting Files
     * @param model String | Model Namespace And Class Name
     * @param data Array   | Data That Should Be Inserted
     * @param rules Array  | Validate Rules
     */
    public static function modelInsert(String $model, Array $data, Array $rules) : Object {
        $validator = self::valid($data, $rules);
        if(!$validator){
            $model = app($model);
            $create = $model::create($data);
            if($create){
                return self::responseHandler($create, 'insert');
            }
        }
        return $validator;
    }

    /**
     * Model Records Updater
     * ? This Method Will Update Record From Given Params
     * TODO: Allow Updating Files
     * @param model String | Model Namespace And Class Name
     * @param data Array   | Data That Should Be Update
     * @param rules Array  | Validate Rules
     */
    public static function modelUpdate(String $model, Array $data, Array $rules) : Object {
        $validator = self::valid($data, $rules);
        if(!$validator){
            $model = app($model);
            $da = $model->find($data['id']);
            if(!$da){
                return self::responseHandler('Record Not Found', 'not_found');
            }
            if($da->update($data)){
                return self::responseHandler($da, 'update');
            }else{
                return self::responseHandler('Update Failed', 'error');
            }
        }
        return $validator;
    }


    /**
     * One Record Fetch
     * ? This Method Will Fetch One Record
     * @param model String  | Model Namespace And Class Name
     * @param id    Integer | Record ID 
     * @param select Array  | List Of Select Columns
     */
    public static function modelShow(String $model, Int $id, Array $select) : Object {
        $m = app($model);
        $data = $m->find($id);

        if($data){
            return self::responseHandler($data->only($select), 'success');
        }

        return self::responseHandler('Record Not Found', 'not_found');
    }


    /**
     * Model Records Delete
     * ? This Method Will Update Record From Given Params
     * TODO: Allow Updating Files
     * @param model String | Model Namespace And Class Name
     * @param id Integer   | ID Of The Record
     */
    public static function modelDelete(String $model, Int $id) : Object {
        $errors = self::valid(['id' => $id], ['id' => 'required']);

        if($errors){
            return $errors;
        }

        $model = app($model);

        $data = $model->find($id);
        if(!$data){
            return self::responseHandler('Record Not Found', 'not_found');
        }

        $data->delete();

        return HelpersController::responseHandler('Record Deleted Successfully', 'success');
    }
}

```
