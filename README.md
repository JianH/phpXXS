phpXXS
======
function RemoveXSS($val) {
06
   $val = preg_replace('/([\x00-\x08,\x0b-\x0c,\x0e-\x19])/', '', $val); 
07
   $search = 'abcdefghijklmnopqrstuvwxyz';
08
   $search .= 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'; 
09
   $search .= '1234567890!@#$%^&*()';
10
   $search .= '~`";:?+/={}[]-_|\'\\';
11
   for ($i = 0; $i < strlen($search); $i++) {
12
      // ;? matches the ;, which is optional
13
      // 0{0,7} matches any padded zeros, which are optional and go up to 8 chars
14
  
15
      // @ @ search for the hex values
16
      $val = preg_replace('/(&#[xX]0{0,8}'.dechex(ord($search[$i])).';?)/i', $search[$i], $val); // with a ;
17
      // @ @ 0{0,7} matches '0' zero to seven times 
18
      $val = preg_replace('/(&#0{0,8}'.ord($search[$i]).';?)/', $search[$i], $val); // with a ;
19
   }
20
  
21
   // now the only remaining whitespace attacks are \t, \n, and \r
22
   $ra1 = Array('javascript', 'vbscript', 'expression', 'applet', 'meta', 'xml', 'blink', 'link', 'style', 'script', 'embed', 'object', 'iframe', 'frame', 'frameset', 'ilayer', 'layer', 'bgsound', 'title', 'base');
23
   $ra2 = Array('onabort', 'onactivate', 'onafterprint', 'onafterupdate', 'onbeforeactivate', 'onbeforecopy', 'onbeforecut', 'onbeforedeactivate', 'onbeforeeditfocus', 'onbeforepaste', 'onbeforeprint', 'onbeforeunload', 'onbeforeupdate', 'onblur', 'onbounce', 'oncellchange', 'onchange', 'onclick', 'oncontextmenu', 'oncontrolselect', 'oncopy', 'oncut', 'ondataavailable', 'ondatasetchanged', 'ondatasetcomplete', 'ondblclick', 'ondeactivate', 'ondrag', 'ondragend', 'ondragenter', 'ondragleave', 'ondragover', 'ondragstart', 'ondrop', 'onerror', 'onerrorupdate', 'onfilterchange', 'onfinish', 'onfocus', 'onfocusin', 'onfocusout', 'onhelp', 'onkeydown', 'onkeypress', 'onkeyup', 'onlayoutcomplete', 'onload', 'onlosecapture', 'onmousedown', 'onmouseenter', 'onmouseleave', 'onmousemove', 'onmouseout', 'onmouseover', 'onmouseup', 'onmousewheel', 'onmove', 'onmoveend', 'onmovestart', 'onpaste', 'onpropertychange', 'onreadystatechange', 'onreset', 'onresize', 'onresizeend', 'onresizestart', 'onrowenter', 'onrowexit', 'onrowsdelete', 'onrowsinserted', 'onscroll', 'onselect', 'onselectionchange', 'onselectstart', 'onstart', 'onstop', 'onsubmit', 'onunload');
24
   $ra = array_merge($ra1, $ra2);
25
  
26
   $found = true; // keep replacing as long as the previous round replaced something
27
   while ($found == true) {
28
      $val_before = $val;
29
      for ($i = 0; $i < sizeof($ra); $i++) {
30
         $pattern = '/';
31
         for ($j = 0; $j < strlen($ra[$i]); $j++) {
32
            if ($j > 0) {
33
               $pattern .= '('; 
34
               $pattern .= '(&#[xX]0{0,8}([9ab]);)';
35
               $pattern .= '|'; 
36
               $pattern .= '|(&#0{0,8}([9|10|13]);)';
37
               $pattern .= ')*';
38
            }
39
            $pattern .= $ra[$i][$j];
40
         }
41
         $pattern .= '/i'; 
42
         $replacement = substr($ra[$i], 0, 2).'<x>'.substr($ra[$i], 2); // add in <> to nerf the tag 
43
         $val = preg_replace($pattern, $replacement, $val); // filter out the hex tags 
44
         if ($val_before == $val) { 
45
            // no replacements were made, so exit the loop 
46
            $found = false; 
47
         } 
48
      } 
49
   } 
50
   return $val; 
51
}
