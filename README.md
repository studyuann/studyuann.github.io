<html><head>
    <meta charset="utf-8">
    <meta http-equiv="x-ua-compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Lostark OpenAPI Developer Portal</title>
    <meta name="robots" content="index,follow">
    <meta name="author" content="Smilegate RPG">
    <meta name="keywords" content="">
    <meta name="description" content="">
    <link rel="shortcut icon" href="https://cdn-lostark.game.onstove.com/2018/obt/assets/images/common/icon/favicon.ico?v=20221109160851">
    <link rel="icon" href="https://cdn-lostark.game.onstove.com/2018/obt/assets/images/common/icon/favicon-192.png?v=20221109160851" sizes="192x192">
    <link rel="stylesheet" href="https://cdn-lostark.game.onstove.com/fonts/notosanskr/v5/notosanskr.css?v=timestamp">
    <link rel="stylesheet" href="https://cdn-lostark.game.onstove.com/developer/assets/css/developer.css?v=timestamp">
    <link rel="stylesheet" type="text/css" href="https://cdn-lostark.game.onstove.com/developer/assets/css/swagger-ui.css">

    <script src="https://cdn-lostark.game.onstove.com/developer/assets/js/developer.js?v=timestamp"></script>
<style type="text/css">@import url('https://fonts.googleapis.com/css2?family=Work+Sans:ital,wght@0,100..900;1,100..900&display=swap');</style></head>
<body>
    

    <div id="wrapper" class="wrapper_developer api_docs">
        
<header>
    <div class="header__wrap">
        <h1><a href="/">Developer</a></h1>
        <nav>
            <ul>
                <li><a href="/getting-started" class="active">API DOCUMENTS</a></li>
                <li><a href="/changelog">CHANGELOG</a></li>
                <li><a href="/faq">FAQ</a></li>
            </ul>
        </nav>
        <div class="header-menu">
            <div class="language_wrap" style="display: none;">
                <div class="language_select">
                    <span>한국어</span>
                </div>
                <div class="lang_list">
                    <div class="lui-select select--lang" data-uid="mf246fde">
                        <div class="lui-select__title" tabindex="0" data-title="한국어">한국어</div>
                        <div class="lui-select__option" role="listbox">
                            <label role="option" tabindex="0" class="lui-select__label--selected"><input type="radio" name="language" data-selected="selected">한국어</label>
                            <label role="option" tabindex="0"><input type="radio" name="language">English (US)</label>
                        </div>
                    </div>
                </div>
            </div>
            <div class="api_status normal">
                <strong>API STATUS</strong>
                <div class="status_txt">All APIs are <em>operational.</em></div>
            </div>
            <div class="my_info">
                <button class="expand-myinfo login">
                    <span>STOVE 219363817</span>
                </button>

                <div class="my_client">
                    <div class="my_client_cont">
                            <div class="apikey_info no_client">
                            </div>
                    </div>
                    <div class="btn_bttm">
                        <a href="/clients" class="btn_go">MY CLIENTS</a>
                        <a href="#" class="btn_logout">LOGOUT</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <div id="notice-display-panel" class="header__notice" style="display: none;">
        <div id="notice-display" class="notice_txt">
            <span></span>
            
        </div>
    </div>
</header>
<style>
    .my_client_show_cust {
        display: block;
    }

    .wrapper_developer header .api_status.maintenance .status_txt, .wrapper_developer header .api_status.maintenance strong {
        background-color: #3e71ff;
    }

    .wrapper_developer header .api_status.maintenance .status_txt em {
        color: #000;
    }

    .my_client.my_client_show {
        z-index: 10;
    }

    #notice-display-panel {
        position: relative;
    }

    #close-notice {
        right: 15px;
        position: absolute;
    }

    #notice-display-panel a {
        color: #fff;
    }
</style>
<script>
    $(document).ready(function () {
        $(".btn_logout").click(function (e) {
            location.href = "https://accounts.onstove.com/logout?" + "&redirect_url=" + encodeURIComponent(location.href);
        });
        var displayer, prevAlarms;
        getApiStatus = () => {
            var polling = {
                method: "GET",
                url: "/api-status",
                contentType: "application/json",
                async: true
            };

            var pollingForAlarms = {
                method: "GET",
                url: "/api-alarm",
                contentType: "application/json",
                async: true
            };

            $.ajax(polling.url, polling).always((data, status, xhr) => {
                var statusElement = $(".api_status");
                var text = statusElement.find(".status_txt");

                if (status === "success") {
                    statusElement.removeClass("normal overload error nodata");
                    text.html("");
                    switch (data.ResultCode) {
                        case 'NO_DATA':
                            statusElement.addClass("nodata");
                            text.html("<em>Syncing...</em>");
                            break;
                        case 'ERROR':
                            statusElement.addClass("error");
                            text.html("<em>An error occurred in the API.</em>");
                            break;
                        case 'SUCCESS':
                            statusElement.addClass("normal");
                            text.html("All APIs are <em>operational.</em>");
                            break;
                        case 'WARNING':
                            statusElement.addClass("overload");
                            text.html("All Response is delayed due to <em>API server overload.</em>");
                            break;
                        case 'MAINTENANCE':
                            statusElement.addClass("maintenance");
                            text.html("<em>Maintenance</em>");
                            break;
                    }
                } else if (status === "error" && data.status === 503) {
                    statusElement.removeClass("normal overload error nodata maintenance");
                    statusElement.addClass("maintenance");
                    text.html("<em>Maintenance</em>");
                } else {

                }
            });

            var contentSet = (data, i) => {
                if (data[i].Contents.includes("{CHANGELOG}")) {
                    $("#notice-display").find("span").html(data[i].Contents.replaceAll("{CHANGELOG}", "<a href='/changelog'>CHANGELOG</a>"));
                } else {
                    $("#notice-display").find("span").text(data[i].Contents);
                }
            };
            $.ajax(pollingForAlarms.url, pollingForAlarms).done(function (data, status, xhr) {                
                if (data.length >= 1) {
                    if (JSON.stringify(prevAlarms) !== JSON.stringify(data)) {
                        var i = data.length - 1;
                        $("#notice-display-panel").show();
                        if (displayer !== undefined && displayer !== null) clearInterval(displayer);

                        contentSet(data, i);
                        displayer = setInterval(() => {
                            i--;
                            if (i < 0) {
                                i = data.length - 1;
                            }
                            contentSet(data, i);
                        }, 1000 * 10);

                        prevAlarms = data;
                    }
                } else {
                    $("#notice-display-panel").hide();
                    if (displayer !== undefined && displayer !== null) clearInterval(displayer);
                }
            }).fail((xhr, status, error) => {                
                $("#notice-display-panel").hide();
                if (displayer !== undefined && displayer !== null) clearInterval(displayer);
            }).always(() => {
            });

            $("#close-notice").click(function (e) {
                $("#notice-display-panel").hide();
            })
        };

        var sleep = (delay) => new Promise(res => setTimeout(res, delay))
        var polling = (task, delay) => task()
            .then(sleep(delay).then(() => polling(task, delay)));
        polling(() => new Promise(getApiStatus), 1000 * 30);

        // Get cookies
        var map = new Map(document.cookie.split('; ').map(v => v.split(/=(.*)/s).map(decodeURIComponent)));

        var expiredAction = () => {
            // init cookie...
            var cookies = document.cookie.split("; ");
            for (var c = 0; c < cookies.length; c++) {
                var d = window.location.hostname.split(".");
                while (d.length > 0) {
                    var cookieBase = encodeURIComponent(cookies[c].split(";")[0].split("=")[0]) + '=; expires=Thu, 01-Jan-1970 00:00:01 GMT; domain=' + d.join('.') + ' ;path=';
                    var p = location.pathname.split('/');
                    document.cookie = cookieBase + '/';
                    while (p.length > 0) {
                        document.cookie = cookieBase + p.join('/');
                        p.pop();
                    };
                    d.shift();
                }
            }
            var username = $(".expand-myinfo").unbind("click");
            username.find("span").text("LOGIN");
            username.click(function (e) {
                e.stopImmediatePropagation();
                location.href = "https://accounts.onstove.com/login?" + "&redirect_url=" + encodeURIComponent(location.href);
            });
        };

        // Check if expired...
        var jwt = map.get("SUAT");
        if ((jwt === undefined || jwt === null || jwt.includes(".") == false)) {
            expiredAction();
        } else {
            // Looks like it is valid...
            var username = $(".expand-myinfo").unbind("click");
            var base64Url = jwt.split('.')[1];
            var base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
            var jsonPayload = decodeURIComponent(window.atob(base64).split('').map(function (c) {
                return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2);
            }).join(''));
            var parsed = JSON.parse(jsonPayload);

            if (parsed.member_no !== undefined || parsed.member_no !== null) {
                if (username !== undefined || username !== null || username.length > 0) {
                    username.find("span").text("STOVE " + parsed.member_no);
                }
            }

            username.click(function (e) {
                $(".my_client").removeClass("my_client_show_cust");

                var check = $(".no_client");
                var popupOpened = $(".my_client").hasClass("my_client_show");

                // if no clients are found...
                if (check.length > 0 && popupOpened == false) {
                    $(".my_client").addClass("my_client_show_cust");

                    var loader = new lui.utils.Loader({
                        parent: $('.apikey_info')
                    });

                    var ajaxOption = {
                        method: "GET",
                        url: "/clients/list",
                        contentType: "application/json",
                        async: true
                    };

                    $.ajax(ajaxOption.url, ajaxOption).always((data, status, xhr) => {
                        if (status === "success") {
                            // invalid session, something went wrong with the client cookie or the platform server.
                            if (typeof (data) === "string") {
                                expiredAction();
                                location.href = "https://accounts.onstove.com/login?" + "&redirect_url=" + encodeURIComponent(location.href);
                            }

                            if (data.length > 0) {
                                var html = '<div class="lui-select select--client">';
                                html += '<div class="lui-select__title" tabindex="0">' + e.ClientName + '</div>';
                                html += '<div class="lui-select__option" role="listbox">';
                                $.each(data, function (i, e) {
                                    html += '<label role="option" tabindex="0"><input type="radio" name="client" data-throttling="' + e.Limit + '" data-key="' + e.Token + '" />' + e.ClientName + '</label>';
                                });
                                html += '</div>';
                                html += '</div>';

                                html += '<div class="apikey_info">';
                                html += '<h2>API KEY</h2>';
                                html += '<button type="button" class="btn_copy">COPY</button>';
                                html += '<p id="api_key"></p>';
                                html += '<h2>RATE LIMIT</h2>';
                                html += '<p id="api_limit"><strong>' + data[0].Limit + '</strong> Requests per minute</p>';
                                html += '</div>';

                                $(".my_client_cont").html(html);

                                var select = new lui.utils.Select({
                                    target: document.querySelector('.my_client_cont .lui-select')
                                });

                                $(".my_client_cont .lui-select__option input[type=radio]").click();

                                check.remove();
                            } else {
                                var html = '<div class="no_client">';
                                html += '<p>There is no created client.<br/>Please create your client.</p>';
                                html += '<a href="/clients/create" class="btn_new">CREATE A NEW CLIENT</a>'
                                html += '</div>';

                                $(".my_client_cont").html(html);
                                $(".my_client .btn_bttm .btn_go").show();
                            }
                        } else if (xhr == "Service Unavailable") {
                            var html = '<div class="no_client">';
                            html += '<p>Client query is not available.<br/>Service went down for maintenance.</p>';
                            html += '</div>';

                            $(".my_client_cont").html(html);
                            $(".my_client .btn_bttm .btn_go").hide();
                        } else {
                            popupOpened = true;
                            expiredAction();
                        }

                        loader.remove();
                    });
                }
            });
        }

        $("#close-notice").click(function (e) {
            $(this).parents(".header__notice").hide();
        });
    });
</script>

        <section class="contents">
            


<aside>
    <div class="nav-container">
        <ul class="lnb">
            <li id="getting-started-item" class="has-depth" data-status="open">
                <a link="/getting-started">GETTING STARTED</a><!--서브 메뉴가 있으면 class="has-depth" data-status="close" / 열리면 class="on", data-status="open"-->
                <!--2 depth-->
                <ul>
                    <li><a href="#login">Login</a></li>
                    <li><a href="#create-client">Create client</a></li>
                    <li><a href="#jwt-key">JWT Key</a></li>
                    <li><a href="#api-doc">API Documentation</a></li>
                    <li><a href="#swagger-auth">Swagger Authorization</a></li>
                    <li><a href="#throttling">Throttling</a></li>
                </ul>
            </li>
            <li id="usage-guide-item" class="has-depth" data-status="close">
                <a link="/usage-guide">USAGE GUIDE</a>
                <ul>
                    <li><a href="#http-basic">Basic HTTP</a></li>
                    <li><a href="#api-request-get-example">GET Example</a></li>
                    <li><a href="#api-request-post-example">POST Example</a></li>
                    <li><a href="#api-errors">API Errors</a></li>
                    <li><a href="#maintenance-error">During maintenance</a></li>
                    <li><a href="#inefficient-api-request">Inefficient API Request</a></li>
                    <li><a href="#api-request-limit">API Request Limit</a></li>
                    <li><a href="#api-status">API Status</a></li>
                    <li><a href="#api-versioning">API Versioning</a></li>
                    <li><a href="#deprecation">Deprecation</a></li>
                </ul>
            </li>
            
                <li class="swagger-item"><a href="#API-NEWS" data-path="NEWS">NEWS</a></li>
                <li class="swagger-item"><a href="#API-CHARACTERS" data-path="CHARACTERS">CHARACTERS</a></li>
                <li class="swagger-item"><a href="#API-ARMORIES" data-path="ARMORIES">ARMORIES</a></li>
                <li class="swagger-item"><a href="#API-AUCTIONS" data-path="AUCTIONS">AUCTIONS</a></li>
                <li class="swagger-item"><a href="#API-MARKETS" data-path="MARKETS">MARKETS</a></li>
                <li class="swagger-item"><a href="#API-GAMECONTENTS" data-path="GAMECONTENTS">GAMECONTENTS</a></li>
        </ul>
    </div>
</aside>
<script>
    $(document).ready(function (e) {
        $("#swagger-ui").hide();

        $(".has-depth ul > li > a").click(function (e) {
            // if swagger-item is selected
            if (!$("#swagger-ui").is(":hidden")) {
                $("#swagger-ui").hide();
                $(".toggle-wrapper").show();
                $(".title").text($(".lnb > li[data-status=open] > a").text());
            }

            var href = $(this).attr("href");
            var pathname = $(this).parents(".has-depth").find("> a").attr("link");
            if (pathname == location.pathname) {
                return;
            } else {
                location.href = location.origin + pathname + href;
            }
        });
    })
</script>

<script>
    $(document).ready(function (e) {
        $("#getting-started-item").attr("data-status", "open");
        $("nav > ul > li > a").eq(0).addClass("active");
    });
</script>

<style>
    .wrapper_developer.api_docs .contents .card {
        margin-bottom: 35px;
    }

    .wrapper_developer.api_docs .contents .card h3 {
        margin-bottom: 10px;
        font-weight: 700;
        color: #fff;
    }

    .title_p {
        margin-bottom: 35px;
    }

    #swagger-ui .view-line-link {
        display: none !important;
    }
</style>
<link rel="stylesheet" href="https://cdn-lostark.game.onstove.com/developer/assets/css/colors.css">

<section class="api_contents">
    <div class="non_api_contents">
        <h2 class="title">GETTING STARTED</h2>
        <div class="toggle-wrapper">
            <p class="title_p">You can start using the Lostark Open API by following basic procedures below to understand how it works.</p>

            <div class="card">
                <h3 id="login">Login</h3>
                <p>
                    Before using any of the Lostark Open APIs, you need to sign in to your Stove.com account. 
                    If you don't have one yet, please <a href="https://accounts.onstove.com/login?&amp;redirect_url=https%3A%2F%2Fdeveloper-lostark.game.onstove.com%2Fgetting-started" id="sign-up-link">sign up here</a>. 
                    There is no additional step required to sign up for the Lostark Open API website; signing up for Stove will suffice. 
                    Once you sign in, you can create a new client immediately.
                </p>
            </div>

            <div class="card">
                <h3 id="create-client">Create A Client</h3>
                <p>
                    Follow these steps to create a client:
                </p>
                <br>
                <ul>
                    <li>1. Click CREATE A NEW CLIENT button or <a href="/clients/create">this link</a></li>
                    <li>
                        2. In the Client Name field, enter a name to identify your client in the list view. This name will only be associated with your account.
                    </li>
                    <li>3. In the Client URL field, enter the URL of the service that will be using this client.</li>
                    <li>4. In the Client Description field, describe what your application will do.</li>
                    <li>5. Read and agree <a target="_blank" href="/agreement">Terms of Use</a> and <a target="_blank" href="https://clause.onstove.com/stove/terms?category=privacy&amp;only_game=Y&amp;game_id=45">Privacy Policy</a>. <b>( These are NOT optional )</b></li>
                    <li>6. After creating your client, you can view the details on the <a href="/clients">MY CLIENTS</a> page or in the popup that appears when you click the red identity button in the top right corner.</li>
                </ul>
            </div>

            <style>
                .responses-table {
                    border: 1px solid #444e63;
                    margin-top: 20px;
                }

                .responses-table .responses-header td {
                    border-right: 1px solid #444e63;
                    border-bottom: 1px solid #444e63;
                    background-color: #353a45;
                    color: #fff;
                }

                .responses-table tbody tr td {
                    background: #1f2125;
                    border-right: 1px solid #444e63;
                    border-bottom: 1px solid #444e63;
                }

                .responses-table td {
                    padding: 10px 20px;
                }

                .responses-table .right-example td {
                    color: #1cebc7!important;
                }
            </style>
            <div class="card">
                <h3 id="jwt-key">JWT Key</h3>
                <p>
                    When you create a client, a JWT is issued immediately.
                    We have an OAuth security layer in place, but you don't need to worry about refreshing expired tokens
                    or performing tedious tasks to request a new access token.

                    <br><br>

                    The issued security token will remain valid indefinitely
                    unless we determine that your key is not secure or the user deletes the client.
                    We strongly recommend storing the JWT token in a secure place, such as a trusted server-side application.

                    <br><br>
                    For example, if you receive a token called "abcdefghijklmnopqrstuvwxyz", you should set the authorization header as follows:
                    </p><thread>
                            </thread><table id="practical-example" class="responses-table">
                        <tbody><tr class="responses responses-header">
                                <td>Example</td>
                                <td>Validity</td>
                            </tr>
                        
                        </tbody><tbody>
                            <tr class="responses right-example">
                                <td>"Authorization: bearer abcdefghijklmnopqrstuvwxyz"</td>
                                <td>O =&gt; VALID</td>
                            </tr>
                            <tr class="responses">
                                <td>"Authorization: abcdefghijklmnopqrstuvwxyz"</td>
                                <td>X =&gt; INVALID (missing "bearer")</td>
                            </tr>
                            <tr class="responses">
                                <td>"Authorization: bearer {abcdefghijklmnopqrstuvwxyz}"</td>
                                <td>X =&gt; INVALID</td>
                            </tr>
                            <tr class="responses">
                                <td>"Authorization: bearer &lt;abcdefghijklmnopqrstuvwxyz&gt;"</td>
                                <td>X =&gt; INVALID</td>
                            </tr>
                            <tr class="responses">
                                <td>"Authorization: bearer{abcdefghijklmnopqrstuvwxyz}"</td>
                                <td>X =&gt; INVALID</td>
                            </tr>
                            <tr class="responses">
                                <td>"Authorization: bearerabcdefghijklmnopqrstuvwxyz"</td>
                                <td>X =&gt; INVALID</td>
                            </tr>
                        </tbody>
                    </table>
                <p></p>
            </div>


            <div class="card">
                <h3 id="api-doc">API Documentation</h3>
                <p>
                    <a class="click-simulator" href="#API-NEWS">The Lostark Open API documentation</a> allows you to interact with
                    the API's resources without actual implementation logic.
                    You can explore what we offer and make actual requests using your JWT. Always ensure you click the
                    "Try it out" button to activate the "Execute" button, which finalizes the request setup, calls the API, and provides the result.

                    <br><br>
                    Exploring the request parameters and checking the responses before implementing the Lostark Open APIs will save you a significant amount of time and effort.
                </p>
            </div>


            <style>
                .mock-modal.modal-ux {
                    background: #464a54;
                    border: 0;
                    border-radius: 10px;
                    max-width: 650px;
                    min-width: 300px;
                    margin: 10px 0px;
                }

                .mock-modal .modal-ux-header {
                    border-bottom: 0;
                }

                .mock-modal .modal-ux-header h3 {
                    font-family: Noto Sans KR,Malgun Gothic,Apple SD Gothic NEO;
                    color: #ff5000 !important;
                    font-size: 20px;
                    text-transform: uppercase
                }

                .mock-modal .modal-ux-header .close-modal svg {
                    fill: #838891
                }

                .mock-modal .modal-ux-inner {
                    padding: 15px 20px;
                }

                .mock-modal .modal-ux-content {
                    padding: 0
                }

                .mock-modal .modal-ux-content h4 {
                    color: #fff;
                    text-transform: uppercase;
                    font-family: Noto Sans KR,Malgun Gothic,Apple SD Gothic NEO;
                    font-size: 16px
                }

                .mock-modal .modal-ux-content p, .modal-ux-content p code {
                    margin: 0
                }

                .mock-modal .modal-ux-content input[type=text] {
                    background: #2c2f36;
                    border: 1px solid #717989;
                    border-radius: 2px;
                    width: 100%;
                    box-sizing: border-box
                }

                .mock-modal .modal-ux-content label {
                    display: block;
                    padding-top: 25px;
                    text-transform: uppercase
                }

                .mock-modal .modal-ux-content .auth-btn-wrapper {
                    display: flex;
                    justify-content: center;
                    padding: 15px 0 0;
                }

                .mock-modal .modal-ux-content .auth-btn-wrapper .btn {
                    width: 182px;
                    height: 35px;
                    font-size: 18px;
                    font-weight: 700;
                    line-height: 35px;
                    background: #696f7d;
                    border-radius: 3px;
                    vertical-align: middle;
                    text-align: center;
                    color: #020308;
                    text-transform: uppercase;
                    padding: 0;
                    border: 0;
                    margin: 0 5px
                }

                /*.mock-modal .modal-ux-content .auth-btn-wrapper .btn:hover {
                    background: #7b8397
                }*/

                .mock-modal .modal-ux-content .auth-btn-wrapper .btn.authorize {
                    background: #c24100;
                    color: #fff
                }

                /*.mock-modal .modal-ux-content .auth-btn-wrapper .btn.authorize:hover {
                    background: #ff5000
                }*/

                .mock-modal input[type=text] {
                    background: #2c2f36;
                    border: 1px solid #717989;
                    border-radius: 2px;
                    width: 100%;
                    color: #fff;
                    box-sizing: border-box;
                    padding: 0 7px;
                    font-weight: 100;
                }



                .mock-auth-wrapper {
                    -ms-flex-pack: start;
                    justify-content: flex-start;
                    margin: 10px 0px;
                }

                .mock-auth-wrapper .btn {
                    border-radius: 3px;
                    box-shadow: 0 1px 2px rgb(0 0 0 / 10%);
                    color: #fff;
                    font-weight: 700;
                    padding: 0 23px 1px;
                    min-height: 30px;
                    transition: all .3s;
                }

                .mock-auth-wrapper .authorize {
                    margin-left: 0
                }

                .mock-auth-wrapper .authorize.btn {
                    border: 0;
                    height: 42px;
                    color: #fff;
                    text-transform: uppercase;
                    font-size: 18px;
                    background: #c24100
                }

                .mock-auth-wrapper .authorize.btn span {
                    padding: 0 0 0 0
                }

                /*.mock-auth-wrapper .authorize.btn:hover {
                    background: #ff5000
                }*/
            </style>
            <div class="card">
                <h3 id="swagger-auth">Swagger Authorization</h3>
                <p>To authorize and call an API in the <a class="click-simulator" href="#API-NEWS">Lostark Open API documentation</a>, you need to pass your JWT as follows:</p>
                <br>
                <ul>
                    <li>1. Click AUTHORIZE button</li>
                    <li>
                        <div class="swagger-ui mock-auth-wrapper">
                            <button class="btn authorize unlocked">
                                <span>Authorize</span>
                            </button>
                        </div>
                    </li>
                    <li>2. Enter your JWT in the VALUE field</li>
                    <li>※ Make sure to avoid <a href="#practical-example">the bad examples</a> presented above for successful validations of your token.</li>
                    <li>
                        <div class="mock-modal modal-ux">
                            <div class="modal-dialog-ux">
                                <div class="modal-ux-inner">
                                    <div class="modal-ux-header"><h3>Available authorizations</h3></div>
                                    <div class="modal-ux-content">
                                        <div class="auth-container">
                                            <div>
                                                <div>
                                                    <h4><code>apiKeyAuth</code>&nbsp;(apiKey)</h4><div class="wrapper">
                                                        <div class="markdown">
                                                            <p>
                                                                Enter your API JWT here to be used with the interactive documentation<br>ex:) bearer JWT<br>
                                                                ※ if unauthorized with your valid token, please review <a target="_blank" href="/getting-started#practical-example">the bad examples.</a><br>
                                                                ※ Ensure you avoid the invalid cases.
                                                            </p>
                                                        </div>
                                                    </div><div class="wrapper"><p>Name: <code>authorization</code></p></div><div class="wrapper"><p>In: <code>header</code></p></div><div class="wrapper"><label>Value:</label><section class=""><input type="text" value="bearer your_JWT_token" disabled=""></section></div>
                                                </div>
                                            </div><div class="auth-btn-wrapper"><button disabled="" class="btn modal-btn auth authorize button">Authorize</button><button disabled="" class="btn modal-btn auth btn-done button">Close</button></div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </li>
                </ul>
            </div>


            <div class="card">
                <h3 id="throttling">Throttling</h3>
                <p>
                    Clients are limited to 100 requests per minute. Exceeding this quota results in a 429 response until the quota is reset. 
                    The quota is automatically renewed every minute, so your application should resume working after a minute once the limit is reached.
                </p>
                <br>
                <h3>Throttling metadata in the response header</h3>
                <p>
                    429 responses are probably meaningless for you and you may feel frustrated. Throttle your client application then! these metadata below will come in handy for you.<br>
                    <b>X-RateLimit-Limit : </b> Request per minute (int)<br>
                    <b>X-RateLimit-Remaining : </b>The number of available requests for your client. (int)<br>
                    <b>X-RateLimit-Reset : </b> The epoch time for the next quota refresh. (long)<br>
                </p>
            </div>
        </div>
    </div>

    <div id="swagger-ui" style="display: none;"></div>
</section>
<script>
    $(document).ready(function (e) {
        var signupLink = "https://accounts.onstove.com/login?" + "&redirect_url=" + encodeURIComponent(location.href);
        $("#sign-up-link").attr("href", signupLink);
    });
</script>

<script src="https://cdn-lostark.game.onstove.com/developer/swagger/swagger-ui-bundle.js" charset="UTF-8"></script>
<script src="https://cdn-lostark.game.onstove.com/developer/swagger/swagger-ui-standalone-preset.js" charset="UTF-8"></script>

<script>
    $(document).ready(function () {
        var li = $(".swagger-item");
        for (var i = 0; i < li.length; i++) {
            li.eq(i).click(function (e) {
                $("#swagger-ui").show();
                $(".swagger-item").removeClass("on");
                $(this).addClass("on");
                var name = $(this).find("a").attr("data-path").toLowerCase();
                var wrapper = $(".non_api_contents");
                if (wrapper !== undefined && wrapper !== null && wrapper.length > 0) {
                    $(".toggle-wrapper").hide();
                    wrapper.find(".title").text(name.toUpperCase());
                }

                window.ui = SwaggerUIBundle({
                    url: "/swagger-doc/endpoints/" + name,
                    dom_id: '#swagger-ui',
                    onComplete: function (e) {
                        var addingButton = () => {
                            var containers = $(".model-container").find("> .model-box");
                            containers.css("position", "relative");
                            $("<button class='button-collapse-model' style='position: absolute; top:10px; right: 10px; background:none; color:#a3b4d8'>Expand Operations</button>")
                                .appendTo(containers)
                                .click(function (e) {
                                    $(this).parent().find("> .model-box-control").click();
                                    var wrapper = $(this).parent().find("> div");
                                    var i = setInterval(() => {
                                        if (wrapper.find("table.model tr").length > 0) {
                                            if (wrapper.find("[aria-expanded='false']").length > 0) {
                                                [wrapper.find("[aria-expanded='false']")].map((el) =>
                                                    el.click()
                                                );
                                            } else {
                                                clearInterval(i);
                                            }
                                        }
                                    }, 10);
                                });
                        }
                        addingButton();
                        $(".models > h4").click(function (e) {
                            setTimeout(() => {
                                if ($(this).parent().hasClass("is-open")) addingButton();
                            }, 50);
                        });
                    },
                    persistAuthorization: true,
                });
            });
        }

        $(document).on("click", ".tablinks[data-name='model']", function (e) {
            $(this).parents(".tab").next().find(".model-box").css("position", "relative");
            var container = $(this).parents(".tab").next().find(".model-box > .model > span");

            $("<button class='button-collapse-model' style='position: absolute; top:10px; right: 10px; background:none; color:#a3b4d8'>Expand Operations</button>")
                .appendTo(container)
                .click(function (e) {
                    $(this).parent().find(".model-box-control[aria-expanded='false']").click();
                    var wrapper = $(this).parent().find("table.model");
                    var i = setInterval(() => {
                        if (wrapper.find("tr").length > 0) {
                            if (wrapper.find("[aria-expanded='false']").length > 0) {
                                [wrapper.find("[aria-expanded='false']")].map((el) =>
                                    el.click()
                                );
                            } else {
                                clearInterval(i);
                            }
                        }                        
                    }, 10);
                });
        });

        $(document).on("click", ".auth-wrapper .authorize", function (e) {
            setTimeout(() => $(".wrapper").find("input[type=text]").val("bearer "), 10);
        });

        //$(document).on("click", ".modal-ux .auth-btn-wrapper .authorize", function (e) {            
        //    var textInput = $(this).parent().prev().find("input[type=text]")
        //    var value = $(this).parent().prev().find("input[type=text]").val();
        //    if (!value.toLowerCase().startsWith("bearer ")) {
        //        value = "bearer " + value;
        //    }

        //    if (value.includes("{") || value.includes("}")) {
        //        value = value.replaceAll("{", "").replaceAll("}", "");
        //    }

        //    textInput.val(value).trigger("keyup");
        //});

        // hash ui trace
        if (location.hash !== null && location.hash !== undefined && location.hash !== '') {
            if (location.hash.startsWith("#API-")) {
                $(".swagger-item").removeClass("on");
                $(".swagger-item").find("a[href='" + location.hash + "']").parent().click();
            } else {
                $(".has-depth ul > li").removeClass("on");
                var anchors = $(".has-depth ul > li > a");
                var target = anchors.filter((i, e) => $(e).attr("href").includes(location.hash));
                $(target).parent().addClass("on");
            }
        }

        $(".click-simulator").click(function(e) {            
            $(".swagger-item").removeClass("on");
            $(".swagger-item").find("a[href='" + $(this).attr("href") + "']").parent().click();
        });
    });
</script>
        </section>

        
<footer>
    <div class="footer__wrap">
        <ul>
            <li><a href="mailto:developer-lostark@smilegate.com">Email us</a></li>
            <li><a target="_blank" href="https://clause.onstove.com/stove/terms?category=privacy&amp;only_game=Y&amp;game_id=45">Privacy Policy</a></li>
            <li><a target="_blank" href="/agreement">Terms of Service</a></li>
        </ul>
        <p>© 2025 Smilegate RPG All rights reserved. Lostark are trademarks, service marks of Smilegate RPG</p>
    </div>
</footer>
<script>
    lui.developer.common();

    $(document).ready(function (e) {

        $(document).on("click", ".my_client_cont .lui-select__option label", function (e) {
            var token = $(this).parent().find(".lui-select__label--selected").find("input[type=radio]").attr("data-key");
            var limit = $(this).parent().find(".lui-select__label--selected").find("input[type=radio]").attr("data-throttling");
            $("#api_key").text(token);
            $("#api_limit").text(limit);

        });

        if ($(".my_client_cont .lui-select").length > 0) {
            // one of the client views... the data exists...
            // populate one
            $(".my_client_cont .lui-select__option label").eq(0).find("input").click();
        }

        // popups
        $(document).on('click', '.layer_wrap .layer_close', function (e) {
            $(this).parents(".layer_wrap").removeClass('show');
            e.preventDefault();
        });
        $(document).on('click', '.layer_wrap .btn_cancel', function (e) {
            $(this).parents(".layer_wrap").removeClass('show');
            e.preventDefault();
        });
        $(document).on('click', '.layer_wrap .btn_ok', function (e) {
            $(this).parents(".layer_wrap").removeClass('show');
            e.preventDefault();
        });
    });
</script>
    </div>

<tldx-lmi-shadow-root data-wxt-shadow-root=""></tldx-lmi-shadow-root><div id="tldx-toast-container"></div></body></html>
