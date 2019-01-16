<input type="button" id="btn_alert" value="Alert" onClick="alert('我是弹出对话框')"/>

rb：
require 'watir'
require 'watir\contrib\enabled_popup'

ie = Watir::IE.new
ie.goto("D:/test.html")

sleep 5
ie.button(:id, 'btn_alert').click_no_wait

hwnd = ie.enabled_popup(10)
w = WinClicker.new

w.clickWindowsButton_hwnd(hwnd, "确定")