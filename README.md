ulSelects
=========

ul下拉框
==============jquery.ulSelects.js===============

(function($) {
  
  /**
  * 扩展的供外部调用的api如下：
  * initSelects(param);
  * setOptionsCSS(optionsCSS);
  * setMaxOptions(optionsLength);
  
  * @author dengjianquan
  * Licensed under the MIT License
  */
  $.extend($.fn, {
    /**
    * 初始化，必须在第一次使用时调用
    * optionValues 必须。下拉元素数据源，类型为数组类型，格式如：
      [
      {'value':'86value1','text':'text1'},
      {'value':'8601value2','text':'text2'},
      ......
      ]//两个属性名称不可更改，必须为：value和text
    * textElement 可选。设置text的目标元素，注意，这里不是元素id，而是元素。如:document.getElementById('id')
    * optionsLength 可选。下拉项长度
    * optionsCSS 可选。下拉项的样式
    * optionSelectedCSS 可选。下拉项选中的样式。
    * callback 可选。回调函数，当选择完成后进行回调
    * isCheck 可选。是否检查选择框中值的合法性,默认不检查
    * 例子：
    *  $('#select').initSelects({
        optionValues : optionsValus,
        'textElement' : document.getElementById('showText'),
        callback : doSomethingFunction
      })
    */
    initSelects : function(params) {
      if(!this.length) {
        return;
      }
     var ulSelects = $.data(this[0],"ulSelects");
      if(!ulSelects) {
        ulSelects = new $.ulSelects(params, this[0]);
        $.data(this[0],'ulSelects',ulSelects);
      }
      return this;
    },
    //设置下拉的样式
    setOptionsCSS : function(optionsCSS) {
      $.data(this[0],"ulSelects").setOptionsCSS(optionsCSS);
      return this;
    },
    //设置最大下拉项
    setMaxOptions : function(optionsLength) {
      $.data(this[0],"ulSelects").setMaxOptions(optionsLength);
      return this;
    }
    
  });
  
  $.ulSelects = function(params,element) {
    //this.params = params;
    this.$element = $(element); //console.info(this.$element);
    this.$textElement = $(params.textElement); //text元素
    this.optionValues = params.optionValues;   //下拉项
    this.setMaxOptions(params.optionsLength);  //下拉长度
    this.setOptionsCSS(params.optionsCSS);     //下拉项样式
    this.setOptionsSelectedCSS(params.optionSelectedCSS); //选项选中样式
    this.callback = params.callback;  //选择后回调函数
    this.isCheck = params.isCheck || false;    //是否检查最后的值是否在下拉项之中，默认为false，不检查
    this.init();
  };
  
 
  $.extend($.ulSelects, {
    //添加u.ulSelect的对象属性
    prototype : {
      init : function() {
        var me = this;
        this.$element.on({
          click : function(e) {
            me.createOptions($(this).val());
          },
          focus : function(e) {
            me.createOptions($(this).val());
          },
          blur : function(e) { //注意，焦点移开后，并没有触发回调。
          	if(me.isCheck) {
            	if(!me.isExist(me.$element.val(),me.optionValues)) { //不合法,置空text和value
            		 me.$element.val('');
			          if(me.$textElement) {
			            me.$textElement.val('');
			          }   
            	}
            }
            me.$element.parent().next().remove();
            me.$element.parent().next().hide();            
          },
          keyup : function(e) {
            var ev = document.all ? window.event : e;
            var $options = me.$element.parent().next(); //下拉项
            var maxIndex = $options.find('li').length;  //下拉项最大索引
            var focusedIndex = parseInt($options.find('li[optionFocused=true]').attr('index'));//当前焦点所在行
            var nextIndex;//下一行
            switch(ev.keyCode) {
              case $.ulSelect.constants.keyCode.TAB: //tab

                break;
              case $.ulSelect.constants.keyCode.ENTER: //enter
                $options.find('li[optionFocused=true]').trigger('mousedown');//enter触发鼠标选中事件
                $options.remove();
                $options.hide();
                break;
              case $.ulSelect.constants.keyCode.UP: //up
                //如果还没焦点，则假定没有生成下拉项，需要重新生成。否则，则焦点往下移
                if(isNaN(focusedIndex)) {
                  me.createOptions($(this).val());
                }else {
                  nextIndex = (focusedIndex-1) >= 0 ?  (focusedIndex-1) : maxIndex-1;
                  me.addFocused($options.find('li[index='+nextIndex+']'));//设新值
                  me.removeFocused($options.find('li[index='+focusedIndex+']'));//去旧值
                }
                break;
              case $.ulSelect.constants.keyCode.DOWN: //down
              	//如果还没焦点，则假定没有生成下拉项，需要重新生成。否则，则焦点往上移
                if(isNaN(focusedIndex)) {
                  me.createOptions($(this).val());
                }else {
                  nextIndex = (1+focusedIndex) <= maxIndex-1 ?  (1+focusedIndex) : 0;
                  me.addFocused($options.find('li[index='+nextIndex+']'));//设新值
                  me. removeFocused($options.find('li[index='+focusedIndex+']'));//去旧值
                }
                break;
              default:
                me.createOptions($(this).val());
                break;
            }    
          }
        });
      },

      //设置下拉项css
      setOptionsCSS : function(optionsCSS) {
      	//将自定义的css的属性并入并覆盖默认css，如果相关属性没有，则采用默认css的属性。
        var newCSS = $.extend({},$.ulSelect.constants.optionsCSSDefault,{'top':this.$element.offset().top+this.$element.height() + 8,'left':this.$element.offset().left}, optionsCSS);
        this.optionsCSS = newCSS;
      },

      //设置最大下拉项
      setMaxOptions : function(optionsLength) {
        this.optionsLength = (optionsLength && !isNaN(optionsLength)) ? optionsLength : 10;
      },

      //设置呗选中的下拉项样式
      setOptionsSelectedCSS : function(optionSelectedCSS) {
        var newCSS = $.extend({},$.ulSelect.constants.optionSelectedCSSDefault, optionSelectedCSS);
        this.optionSelectedCSS = newCSS;
      },


      //将字符串中指定的字符颜色化
      colorValus :  function (value,toColorValues){
        if(!toColorValues || toColorValues === null || toColorValues === '') {
          return value;
        }
        return value.replace(new RegExp(toColorValues,"gm"),'<span style="font-weight:bold;font-style:normal;color:#ff7417;">'+toColorValues+'</span>');
      },

      //获取下拉元素的html
      getOptions : function (optionsValus, optionsLength, lookingVal) {
        if(!optionsValus || optionsValus.length === 0 || !optionsLength || optionsLength === null || optionsLength === 0) {
          return '';
        }
        var options = '<ul class=selectOptionClass>';
        for(var i = 0; i < optionsLength && i < optionsValus.length; i++) {
          var value = optionsValus[i].value;
          var text = optionsValus[i].text;
          var colorValue = this.colorValus(value,lookingVal);
          var colorText = this.colorValus(text,lookingVal);
          options += '<li optionFocused=false index='+i+' value='+value+' text='+text+' style="padding:0 5px;line-height:25px;cursor:default;">' + colorValue + '-' + colorText + '</li>';
        }
        options += '</ul>';
        return options;
      },

      //获取下拉元素对象
      get$Options : function (optionsValus,lookingVal) {
        var me = this;
        var $options = $(me.getOptions(optionsValus,this.optionsLength,lookingVal));   //获取下拉选项的jquery对象             
        $options.css(me.optionsCSS)
        .delegate('li','mousedown',function() {
        	//设置选择后的value和text
          me.$element.val($(this).attr('value'));
          if(me.$textElement) {
            me.$textElement.val($(this).attr('text'));
          }          
          if(me.callback) {
            me.callback.apply(me.$element[0],arguments);//选择后回调自定义函数。
          }
        })
        .delegate('li','mouseover',function() {
          me.removeFocused($options.find('li[optionFocused=true]'));//先将已经存在的去掉
          me.addFocused($(this));        
        })
        .delegate('li','mouseout',function() {
          me.removeFocused($(this));
        });
        //给第一个设置焦点标志
        $options.find('li:first').attr('optionFocused',true).css(me.optionSelectedCSS);
        return $options;
      },

      //添加焦点
      addFocused : function ($option) {
        $option.css(this.optionSelectedCSS)
        .attr('optionFocused',true);//设置焦点标志
      },

      //移除焦点
      removeFocused : function ($option) {
        $option.css('background','').css('color','')
        .attr('optionFocused',false);//去掉焦点标志
      },

      //创建下拉列表
      createOptions : function (val) { 
        var me = this;
        var $options = me.$element.parent().next(); //就下拉项
        var $newOptions; //新下拉项
        if(val === null || val === '') {
          $newOptions = me.get$Options(me.optionValues,val);
        }else {
          $newOptions = me.get$Options(me.find(val,me.optionValues),val);
        }

        if($options.find('li').length > 0) {//已经存在，使用replacewith替换
          if($newOptions.length === 0) {//新拉下为空时特殊处理，删除存在的下拉
            $options.remove();
          }else {
            $options.replaceWith($newOptions);
          }
        }else {//没有存在，使用增加到后面
          me.$element.parent().after($newOptions);
        }    

      },

      //查找指定值
      find : function (val,optionsValus) {
        var founds = [];
        for(var i = 0; i < optionsValus.length; i++) {
          //检索value，也检索text
          if(optionsValus[i].value.toLowerCase().indexOf(val.toLowerCase()) > -1 || optionsValus[i].text.toLowerCase().indexOf(val.toLowerCase()) > -1 ) {
            founds.push(optionsValus[i]);
          }
          if(founds.length >= this.optionsLength) {
            break;
          }
        }
        return founds;
      },
      
      //判断val是否在values之中，全匹配
      isExist : function (val) {
      	if(this.optionValues) {
	      	for(var i = 0; this.optionValues && i < this.optionValues.length ; i++) {
	      		if(val == this.optionValues[i].value) {
	      			return true;
	      		}
	      	}
	      }
      	return false;
      }
    }    
    
   
  });
  
  $.ulSelect = $.ulSelect || {};
  //常量定义  
  $.ulSelect.constants = {
     //默认下拉项样式
    optionsCSSDefault : {
      'display':'block',
      'position':'absolute',
      'background':'#fff',
      'border':'#7f9db9 solid 1px',
      'margin':'-1px 0 0',
      'padding':'5px',
      'list-style':'none',
      'font-size':'12px'
    },
    //默认下拉项选中样式
    optionSelectedCSSDefault : {
      'background' : '#cadeff ',
      'color': '#36c'
    },
    
    //事件keyCode
     keyCode: {
       TAB: 9,
       ENTER: 13,
       DOWN: 40,
       UP: 38
	}
  };
  
  
})(jQuery);






========
例子：

    var optionsValus = [
      {'value':'80value1','text':'text1'},
      {'value':'8001value2','text':'text2'},
      {'value':'800101value3','text':'text3'},
      {'value':'800102value4','text':'text4'},
      {'value':'8002value5','text':'text5'}
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
