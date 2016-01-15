# mailchimp-3.0-php-signup
PHP mailing list signup form using Mailchimp's v3.0 API.

Using Mailchimp v3.0 API, this form lets users type their email, click 'sign up', and have their email address added to your Mailchimp list automatically, without any extra windows popping up or confirmation dialogs. 

HTML/JS code:

```html
<script src="https://code.jquery.com/jquery-1.11.3.min.js"></script>

			<form id="signup" action="index.html" method="get">
			    <input type="hidden" name="ajax" value="true" />
			    <input type="email" name="email" id="email" placeholder="Email address" required/>
			    <input type="submit" id="SendButton" name="submit" value="Sign Up" /> <span id="message"></span>
			</form>

			<script>
			$(document).ready(function() {
			    $('#signup').submit(function() {
			        $("#message").html("Adding your email address...");
			        $.ajax({
			            url: '/assets/php/subscribe.php',
			            type: "POST",
			            data: $('#signup').serialize(),
			            success: function(msg) 
			            {
			                $('#message').html("<i class='fa fa-check'></i> Thanks for signing up!");
			            },
			            error: function(msg) 
			            {
			                $('#message').html("<span style='color: red;'><i class='fa fa-times'></i> Error. Please try again later.</span>");
			                console.log(arguments);
			            }
			        });
			        return false;
			    });
			});
			</script>
```

Relevant PHP file located in '/assets/php/', but here's the code: 

```php
<?php

$data = [
    'email'     => $_POST['email'], 
    'status'    => 'subscribed',
    'firstname' => 'Foo',
    'lastname'  => 'Subscriber'
];

syncMailchimp($data);

function syncMailchimp($data) {
    $apiKey = '***YOUR_API_KEY***';
    $listId = '***YOUR_LIST_ID***';

    $memberId = md5(strtolower($data['email']));
    $dataCenter = substr($apiKey,strpos($apiKey,'-')+1);
    $url = 'https://' . $dataCenter . '.api.mailchimp.com/3.0/lists/' . $listId . '/members/' . $memberId;

    $json = json_encode([
        'email_address' => $data['email'],
        'status'        => $data['status'], // "subscribed","unsubscribed","cleaned","pending"
        'merge_fields'  => [
            'FNAME'     => $data['firstname'],
            'LNAME'     => $data['lastname']
        ]
    ]);

    $ch = curl_init($url);

    curl_setopt($ch, CURLOPT_USERPWD, 'user:' . $apiKey);
    curl_setopt($ch, CURLOPT_HTTPHEADER, ['Content-Type: application/json']);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_TIMEOUT, 10);
    curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'PUT');
    curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
    curl_setopt($ch, CURLOPT_POSTFIELDS, $json);                                                                                                                 

    $result = curl_exec($ch);
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);

    return $httpCode;
}

?>
```
