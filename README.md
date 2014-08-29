ulSelects
=========

ul下拉框


例子：

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

  <input id='select' ></input>
  <input id='showText' class="readonly" readonly></input>
  <input type='button' value='取值' onclick='getvalue()'></input>
