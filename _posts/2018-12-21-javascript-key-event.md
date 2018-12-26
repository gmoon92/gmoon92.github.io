---
layout: post
title: "[JavaScript] input 숫자와 자릿수 제한하기"
tags: [JavaScript, jQuery, keydown, keyup, keypress, change, blur, keyEvent, prototype, call, ime-mode]
categories: [JavaScript]
subtitle: "input 소수점 자릿수 제한하기"
feature-img: "md/img/thumbnail/java-script-logo.png"
thumbnail: "md/img/thumbnail/java-script-logo.png"
excerpt_separator: <!--more-->
sitemap:
display: "false"
changefreq: daily
priority: 1.0
---

<!--more-->

# 직접 만들어 본 라이브러리 - input 입력 숫자 소수점 자릿수 제한

---

- jquery 버전 3.3.1

```
<script src="https://code.jquery.com/jquery-3.3.1.js" integrity="sha256-2Kok7MbOyxpgUVvAk/HJ2jigOSYS2auK4Pfzbm7uH60=" crossorigin="anonymous"></script>
```

``` html
<div id="nk">
	<input type="text" value="" class="digitLimit0.4">
</div>
```

``` javascript
$(document).ready(function(){
   //fn_numberKeyEvents();
   var data = {notSelectors:".monneyType"}
   $("#nk").digitNumber();
});
```

``` javascript
(function($, undefined) {
	"use strict"; // 엄격모드
	var defaults = {
		author  : "MG"
	   ,since   : "2018-12-21"
	   ,project : "digitNumber"
	};

	var nk = $.digitNumber = { version : "1.0" }
	$.fn.digitNumber = function(data) {
		var callFn  = arguments[0] || "";
		var options = arguments[1] || {};

		this.each(function(i, _element) {
			var element = $(_element);
			var dNumber = new DigitNumber(element, callFn, options);
			element.data("digitNumber", dNumber);
			dNumber.render();
		});
	}

	function DigitNumber(element, callFn, options) {
		var t = this;

		//export
		t.render = render;
		t.core   = core;
		t.initSelector = initSelector(element, options);
		
		function render() {
			EventManager.call(t, element);
		}
	}

	function EventManager(element) {
		var t = this;

		//import
		t.core.call(t);
		t.event.call(t);

		//constract
		(function() {
			var selector = t.initSelector;
			setImeMode(selector);
			t.addEvent(selector);
		})();

		//ime-mode:disabled
		function setImeMode(selector) {
			$(selector).css("-webkit-ime-mode", "disabled")
					   .css("-moz-ime-mode", "disabled")
					   .css("-moz-ime-mode","disabled")
					   .css("-ms-ime-mode", "disabled")
					   .css("ime-mode", "disabled");
		}
	}

	function core() {
		var t = this;

		//import
		t.format = foramt;
		t.event = event;
		t.digit = digit;
		t.regexp = regexp;
	}

	function event() {
		var t = this;

		//import
		t.core.call(t);
		t.regexp.call(t);
		t.format.call(t);
		t.digit.call(t);

		//export
		t.addEvent   = addEvent;
		t.disConnect = disConnect;

		function addEvent(selector) {
			fetchEventSource(selector);
		}

		function disConnect(event) {
			event.preventDefault(); 			// 현재 이벤트의 기본 동작을 중단한다.
			event.stopPropagation(); 			// 현재 이벤트가 상위로 전파되지 않도록 중단한다.
			event.stopImmediatePropagation(); 	// 현재 이벤트가 상위뿐 아니라 현재 레벨에 걸린 다른 이벤트도 동작하지 않도록 중단한다.
		}

		function fetchEventSource(selector, option) {
			for ( var i in selector) {
				$(selector[i]).bind("change blur", function(e) {
					t.overLimitNumSlice(this);
					var value = t.decimalComma($(this).val());
					$(this).val(value);
				}).bind("keydown",function(event) {
					event = event || window.event; // chorme, ie 이벤트 구별
					var _key   = event.key
					  , _value = $(this).val();

					//위치
					var _point 	 = t.cursorPosition(this)
					 , _dotPoint = _value.indexOf(".");
					
					//포함여부
					var _dotIncludeFlag = _dotPoint > -1 ? true : false;

					//자릿수 obj
					var _realLimitDigitObj = t.realLimitDigitObj(this) // 현재 
					  , _realPreDigit  = _realLimitDigitObj[0]
					  , _realPostDigit = _realLimitDigitObj[1]

					  , _stdLimitDigitObj = t.stdLimitDigitObj(this) // 기준 자릿수 obj
					  , _stdPreDigit 	  = _stdLimitDigitObj[0] || -1
					  , _stdPostDigit 	  = _stdLimitDigitObj[1] || -1;

					if (_key == "Tab"
							|| _key == "ArrowRight"
							|| _key == "ArrowLeft"
							|| _key == "Backspace"
							|| _key == "Delete"
							|| _key == "Home"
							|| _key == "End") {
						return;
					} else {
						if (!t.getRegexp("dotAndOnlyNumber").test(_key)) {
							t.disConnect(event);
							return false;
						}

						if (_dotIncludeFlag && _key == ".") {
							t.disConnect(event);
							return false;
						}
						//console.log(_realPostDigit == _stdPostDigit)
						if (_stdLimitDigitObj !== -1) {
							if (_dotIncludeFlag) { //dot include
								if (_point > _dotPoint) {
									//dot post
									if (_realPostDigit == _stdPostDigit) {
										t.disConnect(event);
										return false;
									}
								} else {
									//dot pre
									if (_realPreDigit > _stdPreDigit) {
										t.disConnect(event);
										return false;
									}
								}
							} else {
								//not dot
								if (_realPreDigit == _stdPreDigit) {
									if (_key != ".") {
										t.disConnect(event);
										return false;
									}
								} else {
									if (_realPreDigit > _stdPreDigit) {
										t.disConnect(event);
										return false;
									}
								}
							}

						}
					}
					return;
				});
			}
		}
	}

	function regexp(_type) {
		var t = this;
		t.getRegexp = (function(_type) {
			switch (_type) {
			case "onlyNumber": //only number
				return /^[0-9]*$/;
				break;
			case "dotAndOnlyNumber": // number or dot
				return /^[0-9]*$|\./;
				break;
			case "stdDigitClass": // stdDigitClass
				return /digitLimit/;
				break;
			}
		});
	}

	function foramt() {
		var t = this;

		t.decimalComma = decimalComma;

		function decimalComma(_x) {
			var _x = _x.toString().replace(/[^(0-9|\.)]/gi, "");
			var _preNum = "", _postNum = "";

			var _realObj = _x.toString().split(".");
			if (_realObj.length > 0) {
				_preNum = _realObj[0] || "";
				_preNum = comma(_preNum);
				_postNum = _realObj[1] || "";

				if (_postNum !== "") {
					_postNum = Number("0." + _postNum) + "";
					_postNum = _postNum.substring(2, _postNum.length);
				}
			}

			var dot = _postNum == "" ? "" : ".";
			return _preNum + dot + _postNum;
		}

		function comma(_x) {
			_x = decomma(_x);
			return _x.replace(/\B(?=(\d{3})+(?!\d))/g, ",");
		}

		function decomma(_x) {
			return _x.toString().replace(/[^(0-9)]/gi, "");
		}
	}

	function digit() {
		var t = this;

		//export
		t.cursorPosition = cursorPosition; // 현재 자릿수 반환
		t.overLimitNumSlice = overLimitNumSlice; // 자릿수 넘어가면 자르기
		t.realLimitDigitObj = realLimitDigitObj; // 현재 자릿수 obj
		t.stdLimitDigitObj = stdLimitDigitObj; // 기준 자릿수 obj

		function cursorPosition(_selector) {
			_selector = $(_selector);
			var start = _selector[0].selectionStart, end = _selector[0].selectionEnd, diff = end
					- start;

			/* 
			 if (start >= 0 && start == end) {
			     // do cursor position actions, example:
			     //'Cursor Position: ' + start
			 } else if (start >= 0) {
			     // do ranged select actions, example:
			     //'Cursor Position: ' + start + ' to ' + end + ' (' + diff + ' selected chars)'
			 }
			 */
			return start;
		}

		function overLimitNumSlice(selector) {
			var _realLimitDigitObj = this.realLimitDigitObj(selector), _stdLimitDigitObj = this
					.stdLimitDigitObj(selector);

			if (_realLimitDigitObj.length > 0
					&& _stdLimitDigitObj.length > 0) {

				if (_realLimitDigitObj[0] > _stdLimitDigitObj[0]) {
					var _value = $(selector).val();
					_value = _value.toString().replace(/[^(0-9)|\.]/gi, "");
					_value = _value.substring(0, _stdLimitDigitObj[0]);
					$(selector).val(_value);
				}
			}
		}

		function realLimitDigitObj(_selector) {
			var _value = $(_selector).val();
			_value = _value.toString().replace(/[^(0-9)|\.]/gi, "");

			var _preNumb = 0, _postNumb = 0;

			var _obj = _value.split(".");
			if (_obj.length > 0) {
				var _proObj = _obj[0] || [];
				var _postObj = _obj[1] || [];

				_preNumb = _proObj.length; // 정수 자릿수
				_postNumb = _postObj.length; // 소수점 자릿수
			}

			return [ _preNumb, _postNumb ];

		}

		function stdLimitDigitObj(_selector) {
			var _classList = $(_selector)[0].classList;
			var _limitDigit = -1;

			var _classPattern = t.getRegexp("stdDigitClass");
			for (var i = 0, objLength = _classList.length; i < objLength; i++) {
				var _className = _classList[i];

				if (_classPattern.test(_className)) {
					_limitDigit = _className.replace(_classPattern, "");
				}
			}

			if (_limitDigit === -1) {
				return _limitDigit;
			} else {
				return _limitDigit.split(".");

			}
		}
	}

	function initSelector(element, options) {
		var _reObj = new Array();

		if (element !== undefined) {
			if (element[0].id == "") {
				console.error("ERROR :: Should must you have to ID, not a class name. \nex) $(id).digitNumber(..);");
				return _reObj;
			}

			var inputSeletors = $("#" + element[0].id + " input");

			var notSeletors = options.notSelectors || "";
			notSeletors = notSeletors.replace(/ /gi, "");

			var _notObj = notSeletors.split(",");

			if (_notObj.length > 0) {
				for (var i = 0, tot = inputSeletors.length; i < tot; i++) {
					var include = true;

					for ( var idx in _notObj) {
						var selector = _notObj[idx].substring(0, 1) || "";
						if (selector !== "") {
							var _nS = _notObj[idx].replace(new RegExp("\\"
									+ selector), "");
							var _nSRegex = new RegExp(
									"\\b(" + _nS + ")\\b", "g");

							var selectorStr;
							switch (selector) {
							case "#": // id
								selectorStr = inputSeletors[i].id;
								break;

							case ".": // class
								selectorStr = inputSeletors[i].classList
										.toString();
								break;
							}

							if (_nSRegex.test(selectorStr)) {
								include = false;
								break;
							}
						}
					}

					if (include) {
						_reObj.push(inputSeletors[i]);
					}

				}
			}
		}

		return _reObj;
	}
})(jQuery);
```
