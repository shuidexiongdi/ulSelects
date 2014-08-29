ulSelects
=========

ul下拉框


例子：
<!DOCTYPE html>
<html>
<head>
<style>
INPUT.readonly {
	BORDER-BOTTOM: medium none; BORDER-LEFT: medium none; WIDTH: 200px; BACKGROUND: none transparent scroll repeat 0% 0%; FONT-SIZE: 10pt; BORDER-TOP: medium none; BORDER-RIGHT: medium none
}
</style>
<script src="http://code.jquery.com/jquery-1.9.1.min.js"></script>
<script src="jquery.ulSelects.js"></script>
  <script language='javascript'>
    var optionsValus = [
      {'value':'86value1','text':'text1'},
      {'value':'8601value2','text':'text2'},
      {'value':'860101value3','text':'text3'},
      {'value':'860102value4','text':'text4'},
      {'value':'8602value5','text':'text5'}
    ];
        
    $(document).ready(function () {
      //ulSelects('select',optionsValus,'showText');
      $('#select').initSelects({
        optionValues : optionsValus,//必选
        'textElement' : document.getElementById('showText'), //可选
        'optionsLength' : 10, //可选
        callback : getvalue //可选
      });
    });
    
    
    
//test
 function getvalue() {
    console.info($('#select').val());
   if($('#select').val()) { //some
    
     console.info('22222');
   }
  }
    </script>
  <meta charset="utf-8">
</head>
<body>
  <input id='select' ></input>
  <input id='showText' class="readonly" readonly></input>
  <br>
  <input >
<input type='button' value='取值' onclick='getvalue()'></input>
</body>
</html>
