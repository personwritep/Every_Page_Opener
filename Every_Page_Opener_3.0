// ==UserScript==
// @name        Every Page Opener 💢
// @namespace        http://tampermonkey.net/
// @version        3.0
// @description        「記事の編集・削除」ページで全ての「下書き」を「公開記事」に変更する
// @author        Ameba Blog User
// @match        https://blog.ameba.jp/ucs/entry/srventrylist*
// @match        https://blog.ameba.jp/ucs/entry/srventryupdate*
// @run-at        document-start
// @grant        none
// @updateURL        https://github.com/personwritep/Every_Page_Opener/raw/main/Every_Page_Opener.user.js
// @downloadURL        https://github.com/personwritep/Every_Page_Opener/raw/main/Every_Page_Opener.user.js
// ==/UserScript==


let retry=0;
let interval=setInterval(wait_target, 1);
function wait_target(){
    retry++;
    if(retry>100){ // リトライ制限 100回 0.1secまで
        clearInterval(interval); }
    let target=document.documentElement; // 監視 target
    if(target){
        clearInterval(interval);
        style_in(); }}

function style_in(){
    let style=
        '<style id="EPO">'+
        '#globalHeader, #ucsHeader, #ucsMainLeft h1, #ucsMainRight, .l-ucs-sidemenu-area, '+
        '.selection-bar { display: none !important; } '+

        '#ucsContent { width: 930px !important; } '+
        '#ucsContent::before { display: none; } '+
        '#ucsMain { background: none; } '+
        '#ucsMainLeft { width: 930px !important; padding: 0 15px !important; } '+

        '#entryMonth li a:visited { color: #3970B5 !important; }'+
        '#nowMonth { color: #000; } '+
        '#entryListEdit form { display: flex; flex-direction: column; } '+
        '#entrySort { order: -2; margin-bottom: 2px; } '+
        '#sorting { font-size: 15px; margin: 36px 0 4px; padding: 2px 0; height: 114px; } '+
        '#sorting select, #sorting ul { display: none; } '+
        '.pagingArea { order: -1; '+
        'margin-bottom: -33px; position:unset !important; background: #ddedf3; } '+
        '.pagingArea a { border: 1px solid #888; } '+
        '.pagingArea .active{ border: 2px solid #0066cc; } '+
        '.pagingArea a, .pagingArea .active, .pagingArea .disabled { '+
        'font-size: 14px; line-height: 23px; } '+

        '#sorting input { font-family: meiryo; font-size: 15px }'+
        '#div0 { color: #333; margin: 10px -10px 0 15px; }'+
        '#div1 { color: #000; margin: 8px 15px; border: 1px solid #888; background: #fafcfd; }'+
        '#start_button { padding: 4px 8px 2px; margin: 7px 35px 7px 0; width: 170px }'+
        '#import_sw { padding: 4px 0 2px; margin: 7px 10px 7px 0; width: 130px; }'+
        '#import { display: none; }'+
        '#file_time { display: inline-block; color: #000; padding: 3px 0 2px 12px; }'+
        '#snap_result { display: inline-block; margin: 6px 12px 4px; white-space: nowrap; }'+
        '#am_button { display: inline-block; margin: 6px 15px 0 -20px; float: right; '+
        'cursor: pointer; }'+
        '</style>';

    if(!document.querySelector('#EPO')){
        document.documentElement.insertAdjacentHTML('beforeend', style); }

} // style_in()




window.addEventListener('load', function(){ // 孫ウインドウだけで働く
    let body_id=document.body.getAttribute('id');
    if(body_id=="entryCreate"){ // 孫ウインドウ
        window.document.body.style.background='#c5d8e1';
        window.document.body.style.boxShadow='0 0 0 100vh #c5d8e1';
        window.document.querySelector('#subContentsArea').style.display='none';

        select_e(close_w);

        function select_e(close_w){
            let error_report=document.querySelector('h1.p-error__head');
            if(error_report==null){ // エラー無い場合 grayを送信　子ウインドウを閉じる
                if(window.opener){
                    report('gray');
                    window.opener.close(); }}
            else{ // エラー報告の場合 redを送信　子ウインドウを残す
                if(window.opener){
                    report('red');
                    window.opener.location.reload(); }}
            close_w(); }

        function close_w(){
            window.open('about:blank','_self').close(); } // 孫ウインドウは常に閉じる

        function report(color){
            window.opener.document.querySelector('html').style.color=color; }}
});




window.addEventListener('load', function(){ // 親ウインドウで働くメインスクリプト
    let next;
    let mode; // 動作モード
    let except_am; //「アメンバー記事」以外を開く特別モード
    let mode_arr=[];
    let blogDB={}; // 記事IDリスト
    let entry_id_DB=[]; // ID検索用の配列
    let filter;
    let regex_id;
    let entry_target;
    let entry_id;
    let publish_f;
    let pub_all;
    let pub_dra;
    let pub_ame;
    let list_bar;
    let new_win;
    let link_target;


    let read_json=localStorage.getItem('blogDB_back'); // ローカルストレージ 保存名
    blogDB=JSON.parse(read_json); // blogDB（SNAP記録）はこのスクリプトでは読取り専用
    if(blogDB==null){
        blogDB=[['00000000000', 's']]; }
    else{ reg_set(); }


    function reg_set(){
        let k;
        entry_id_DB=[]; // リセット
        pub_all=0;
        pub_dra=0;
        pub_ame=0;
        for(k=0; k<blogDB.length; k++){
            entry_id_DB[k]=blogDB[k][0]; // ID検索用の配列を作成
            if(blogDB[k][1]=='0'){
                pub_all +=1; continue; }
            if(blogDB[k][1]=='1'){
                pub_dra +=1; continue; }
            if(blogDB[k][1]=='2'){
                pub_ame +=1; continue; }}
        filter=entry_id_DB.join('|');
        regex_id=RegExp(filter); } // フィルター作成


    read_json=localStorage.getItem('blogDB_mode'); // ローカルストレージ 保存名
    mode_arr=JSON.parse(read_json);
    if(!mode_arr || mode_arr.length!=3){
        mode_arr=[0, 0, 0]; }
    mode=mode_arr[0]; // modeが「3」ならページングでぺージを開いた連続作業とみなす
    except_am=mode_arr[2]; //「下書き記事」を全て公開する特別モード
    mode_arr[0]=0; // 不正規の終了を考慮してリセット
    let write_json=JSON.stringify(mode_arr);
    localStorage.setItem('blogDB_mode', write_json); // ローカルストレージ保存


    paging_watch();

    function paging_watch(){
        let k;
        let ym_pager=document.querySelectorAll('#entrySort a');
        for(k=0; k<ym_pager.length; k++){
            ym_pager[k].addEventListener('mousedown', function(){
                mode_send(); }); }
        let pn_pager=document.querySelectorAll('.pagingArea a');
        for(k=0; k<pn_pager.length; k++){
            pn_pager[k].addEventListener('mousedown', function(){
                mode_send(); }); }

        function mode_send(){
            if(mode>0){
                mode_arr[0]=3; //「0」以外から移動時の場合は「3」を保存
                let write_json=JSON.stringify(mode_arr);
                localStorage.setItem('blogDB_mode', write_json); }}} // ローカルストレージ 保存


    entry_target=document.querySelectorAll('.entry-item .entry');
    entry_id=document.querySelectorAll('input[name="entry_id"]');
    publish_f=document.querySelectorAll('input[name="publish_flg"]');
    list_bar=document.querySelectorAll('#entryList .entry-item');



    let body_id=document.body.getAttribute('id');
    if(body_id=='entryListEdit'){ // 親ウインドウでのみ働く
        let box=document.querySelector('#sorting');
        if(box){

            let insert_div=
                '<div id="div0">'+
                '<input id="start_button" type="submit">'+
                '<input id="import_sw" type="button" value="ファイル読込み">'+
                '<input id="import" type="file">'+
                '<span id="file_time">　　</span>'+
                '</div>'+
                '<div id="div1">'+
                '<span id="snap_result"></span>'+
                '<span id="am_button"></span>'+
                '</div>';

            if(!box.querySelector('#div0')){
                box.insertAdjacentHTML('beforeend', insert_div); }


            let button1=box.querySelector('#start_button');
            let button2=box.querySelector('#import');
            let button3=box.querySelector('#import_sw');
            let span3=box.querySelector('#file_time');
            let span7=document.querySelector('#snap_result');
            let span8=box.querySelector('#am_button');







            button1.value='公開処理の開始　▶';
            button1.onclick=function(e){
                e.preventDefault();
                if(mode==0){ // 最初の起動直後
                    if(except_am==0){ // 特別オープンモードOFF
                        start_select(); }
                    else if(except_am==1){ // 特別オープンモードON
                        start_select_ex(); }}
                else if(mode==1){ // 続行状態
                    mode=2; // クリックされたら停止モード
                    button1.value='公開処理を続行　▶'; }
                else if(mode==2){ // 停止状態
                    mode=1; // クリックされたら続行モード
                    button1.value='公開処理を停止　❚❚';
                    open_win(next); }
                else if(mode==3){ // 前ページから連続作業で開いた状態
                    if(entry_target.length==0 || entry_target==null){ // 編集対象がリストに無い場合
                        alert('このページに編集対象の記事がありません'); } // クリックで modeは「3」のまま
                    else if(entry_target.length >0){ // 編集対象がリストに有る場合
                        mode=1; // クリックされたら続行モード
                        button1.value='公開処理を停止　❚❚';
                        open_win(0); }}} // 連続作業でスタート


            function start_select(){
                if(blogDB.length==1){
                    alert(['⛔　 SNAPデータを blogDB(n).json ファイルから読込んでください。',
                           '\n　　　Every Page Snap 💢 で作成した適切なファイルが無い場合は',
                           '\n　　　Every Page Opener  を使用して公開処理をしてください。',
                           '\n\n⛔　〔注意〕',
                           '\n　　　Every Page Opener は 元 「アメンバー記事」 から 「下書き」 に',
                           '\n　　　変更処理をした記事に関しても 「全員に公開」 に変更します。',
                           '\n　　　この点には注意して下さい。　このツールを終了します'].join('') ); }
                else if(entry_target.length==0 || entry_target==null){ // 編集対象がリストに無い場合
                    alert('このページに編集対象の記事がありません'); }
                else if(entry_target.length >0){ // 編集対象がリストに有る場合
                    let conf_str=['🟠　このページの 「下書き」 を SNAPデータに従って公開処理します\n',
                                  '\n　　　　SNAP時の 「全員公開」の記事  ⇨ 「全員に公開」',
                                  '\n　　　　SNAP時の 「アメンバー記事」  ⇨ 「アメンバー限定公開」',
                                  '\n　　　　現在の 「アメンバー限定公開」 は 変更しません',
                                  '\n　　　　SNAPデータに記録がない記事は 変更しません \n',
                                  '\n　　　　　　　　OKを押すと公開処理を開始します'].join('');
                    let ok=confirm(conf_str);
                    if(ok){
                        mode=1;
                        button1.value='公開処理を停止　❚❚';
                        open_win(0); }}}


            function start_select_ex(){
                if(entry_target.length==0 || entry_target==null){ // 編集対象がリストに無い場合
                    alert('このページに編集対象の記事がありません'); }
                else if(entry_target.length >0){ // 編集対象がリストに有る場合
                    let conf_str=['🟠　このページの 「下書き」 を SNAPデータなしで公開処理します\n',
                                  '\n　　　　元の 「全員公開」「アメンバー記事」 ⇨ 「全員に公開」',
                                  '\n　　　　現在の 「アメンバー記事」「全員公開」 は変更しません\n',
                                  '\n　　　　　　　　OKを押すと公開処理を開始します'].join('');
                    let ok=confirm(conf_str);
                    if(ok){
                        mode=1;
                        button1.value='公開処理を停止　❚❚';
                        open_win(0); }}}


            if(except_am==0){ // 特別オープンモードOFFの場合のみ表示
                button3.onclick=function(e){
                    e.preventDefault();
                    button2.click(); }

                button2.addEventListener("change", function(){
                    if(!(button2.value)) return; // ファイルが選択されない場合
                    let file_list=button2.files;
                    if(!file_list) return; // ファイルリストが選択されない場合
                    let file=file_list[0];
                    if(!file) return; // ファイルが無い場合

                    let file_reader=new FileReader();
                    file_reader.readAsText(file);
                    file_reader.onload=function(){
                        if(file_reader.result.slice(0, 15)=='[["00000000000"'){ // blogDB.jsonの確認
                            let data_in=JSON.parse(file_reader.result);
                            blogDB=data_in; // 読込み上書き処理
                            let write_json=JSON.stringify(blogDB);
                            localStorage.setItem('blogDB_back', write_json); // ローカルストレージ 保存
                            reg_set();
                            time_disp(new Date(file.lastModified));
                            snap_disp(); }
                        else{
                            alert("   ⛔ 不適合なファイルです  blogDB(n).json ファイルを選択してください");}};});

                function time_disp(time_obj){
                    function digits_two(n){
                        if(n<10){
                            return '0'+n.toString(); }
                        else{
                            return n.toString(); }}

                    let date_sty=
                        time_obj.getFullYear() + "/" + (time_obj.getMonth() + 1) + "/" + time_obj.getDate() + "　" +
                        digits_two(time_obj.getHours()) + ":" + digits_two(time_obj.getMinutes());
                    span3.textContent=date_sty; }

                snap_disp(); } // if(except_am==0)


            function snap_disp(){
                reg_set();
                let span7=document.querySelector('#snap_result');
                span7.innerHTML='　記録件数：<b>' + (blogDB.length -1) + '</b>　　全員に公開：<b>' + pub_all +
                    '</b>　　アメンバー限定公開：<b>' + pub_ame + '</b>　　下書き：<b>' + pub_dra; +'</b>'; }


            if(except_am==0){ // 特別オープンモードOFF
                span8.textContent='SNAP条件公開 💢'; }
            else if(except_am==1){ // 特別オープンモードON
                span8.textContent='無条件公開 ⚪';
                span7.textContent=
                    '　◼◼◼ 「下書き」を全て「公開」に変更します（現在のアメンバー記事は変更しません）◼◼◼'; }


            span8.onclick=function(e){
                e.preventDefault();
                if(except_am==0){ // 特別オープンモードOFF
                    except_am=1; }
                else if(except_am==1){ // 特別オープンモードON
                    except_am=0; }
                mode_arr[2]=except_am;
                write_json=JSON.stringify(mode_arr);
                localStorage.setItem('blogDB_mode', write_json); // ローカルストレージ保存
                location.reload(); } // 特別オープンモードの変更時はリロードする

        }} // if(body_id=='entryListEdit')



    function open_win(k){
        new_win=Array(entry_target.length);
        link_target=Array(entry_target.length);

        link_target[k]='/ucs/entry/srventryupdateinput.do?id='+ entry_id[k].value;
        let top_p=100 + 30*k;

        if(mode==1 && except_am==0){ // 特別オープンモードOFF
            let index=entry_id_DB.indexOf(entry_id[k].value);
            if(index==-1){ // IDが SNAPデータに記録がない記事の場合
                list_bar[k].style.backgroundColor='#fff5aa';
                next_do(k); } //⏩ 子ウインドウを開かず リスト背景 黄色
            else{ // IDがSNAPデータに記録されていた場合
                if(publish_f[k].value==1){ // リスト上の公開設定が「下書き」のみ処理
                    if(blogDB[index][1]==1){ // SNAPデータが「下書き」の場合
                        list_bar[k].style.backgroundColor='#eceff1';
                        next_do(k); } //⏩ 子ウインドウを開かず リスト背景 淡グレー
                    else{
                        publish_f[k].value=blogDB[index][1]; // リスト上の公開設定をSNAPデータに従って更新する
                        new_win[k]=window.open(link_target[k], k,
                                               'top=' + top_p + ', left=100, width=600, height=180'); // 編集画面を開く
                        let publish_f_status=list_bar[k].querySelector('.status-text');
                        publish_f_status.style='opacity: 0';
                        list_bar[k].style.boxShadow='inset 0 0 0 2px #03a9f4'; // リスト　青枠表示
                        new_win[k].addEventListener('load', function(){
                            setTimeout( function(){
                                edit_target(publish_f[k].value, k); }, 50); });}} //🟦 子ウインドウで書込み開始
                else{ //  リスト上の公開設定が「下書き」ではない場合
                    list_bar[k].style.backgroundColor='#eceff1';
                    next_do(k); }}} //⏩ 子ウインドウを開かず リスト背景 淡グレー

        if(mode==1 && except_am==1){ // 特別オープンモードON
            if(publish_f[k].value==1){ // リスト上の公開設定が「下書き」のみ処理
                publish_f[k].value=0; //「全員に公開」に設定更新する
                new_win[k]=window.open(link_target[k], k,
                                       'top=' + top_p + ', left=100, width=600, height=180'); // 編集画面を開く
                let publish_f_status=list_bar[k].querySelector('.status-text');
                publish_f_status.style='opacity: 0';
                list_bar[k].style.boxShadow='inset 0 0 0 2px #03a9f4'; // リスト　青枠表示
                new_win[k].addEventListener('load', function(){
                    setTimeout( function(){
                        edit_target(publish_f[k].value, k); }, 50); });} //🟦 子ウインドウで書込み開始
            else{ //  リスト上の公開設定が「下書き」ではない場合
                list_bar[k].style.backgroundColor='#eceff1';
                next_do(k); }}}



    function edit_target(val,k){
        new_win[k].addEventListener('beforeunload', flag_line, false);
        publish_do(val,k);

        function flag_line(){
            if(new_win[k]){
                var send_color=new_win[k].document.querySelector('html').style.color; }
            if(send_color=='gray'){ // 文書保存が正常終了した場合
                list_bar[k].style.boxShadow='none'; // 青枠を消す
                list_bar[k].style.backgroundColor='#a4d5fd'; // 子ウインドウが閉じられ　リスト背景 淡ブルー
                let publish_f_status=list_bar[k].querySelector('.status-text');
                if(publish_f[k].value==0){ //「全員に公開」へ処理した場合
                    let sty='color: #000; background: #fff !important; opacity: 1';
                    publish_f_status.textContent='全員に公開';
                    publish_f_status.style=sty; }
                else if(publish_f[k].value==2){ //「アメンバー限定公開」へ処理した場合
                    let sty='color: #fff; background: #009688 !important; opacity: 1; text-indent: 2px';
                    publish_f_status.textContent='アメンバー';
                    publish_f_status.style=sty; }
                next_do(k); } // ⏩
            else if(send_color=='red'){ // 文書保存が異常終了の場合は子ウインドウは閉じない
                list_bar[k].style.boxShadow='inset 0 0 0 2px red'; // リスト 背景 白　赤枠表示
                list_bar[k].style.backgroundColor='#fff';
                next_do(k); }} // ⏩


        function publish_do(val, k){
            let publish_b0=new_win[k].document.querySelector('button.js-submitButton[publishflg="0"]');
            let publish_b1=new_win[k].document.querySelector('button.js-submitButton[publishflg="1"]');
            if(val==0){ publish_b0.click(); }
            if(val==1){ publish_b1.click(); }
            if(val==2){
                let amb_ck=new_win[k].document.querySelector('#amemberFlg');
                amb_ck.checked = true;
                setTimeout(()=>{
                    publish_b0.click();
                }, 20); }}}



    function next_do(k){
        next=k+1;
        if(next<entry_target.length){ open_win(next); }
        else{
            mode=4; // 終了状態
            let button1=document.querySelector('#start_button');
            button1.value='🔵 処理が終りました'; }}

});
