# How to Solve reCAPTCHA with Puppeteer in PHP for Web Scraping

![](https://assets.capsolver.com/prod/images/post/2024-08-27/5dd5d654-d3f6-4e0d-85ee-77b97288355a.png)

Ever hit a wall with reCAPTCHA while scraping data? I’ve been there too. Those CAPTCHA challenges can turn a simple scraping task into a major hurdle. But don’t worry—I’ve got a solution that’ll help you breeze past those barriers.

In this blog, I’ll lead you through using Puppeteer, a powerful Node.js library, to tackle reCAPTCHA challenges. We’ll then integrate this with PHP to make your web scraping tasks smoother and more efficient. Ready to tackle reCAPTCHA and get your data seamlessly? Let’s get in!

### What is reCAPTCHA When Web Scraping?

To give you some context, reCAPTCHA is a system designed to protect websites from automated abuse. It asks users to complete tasks that are easy for humans but challenging for bots, like identifying objects in images or checking a box. While these challenges are great for security, they can be a real pain for web scraping. Here are the basic types you might encounter:

- **[reCAPTCHA v2](https://www.capsolver.com/products/recaptchav2/?utm_source=official&utm_medium=blog&utm_campaign=recaptchaphp)**: This version is known for the “I’m not a robot” checkbox and image-based challenges. Users might need to click on images or complete specific actions to prove they’re human. It’s effective at distinguishing between real users and bots.

![](https://assets.capsolver.com/prod/images/post/2024-08-27/4266ec7d-8bf3-481b-a1ab-f8d133cfc8df.gif)


- **[reCAPTCHA v3](https://www.capsolver.com/products/recaptchav3/?utm_source=official&utm_medium=blog&utm_campaign=recaptchaphp)**: This version operates in the background. Instead of requiring user interaction, it analyzes user behavior across the site and assigns a risk score. This score helps websites determine whether to grant or block access based on the likelihood of the user being a bot.

- **reCAPTCHA Enterprise**: For businesses with more demanding security requirements, reCAPTCHA Enterprise offers advanced protection against sophisticated threats. It includes features like enhanced risk analysis, customizable scoring, and improved scalability, making it suitable for organizations dealing with sensitive information or critical operations.

> Struggling with the repeated failure to completely solve the irritating captcha?
>
> Discover seamless automatic captcha solving with **Capsolver** AI-powered Auto Web Unblock technology!
>
> Claim Your   <u>**Bonus Code**</u> for top captcha solutions; [CapSolver](https://www.capsolver.com/?utm_source=official&utm_medium=blog&utm_campaign=recaptchaphp): **WEBS**. After redeeming it, you will get an extra 5% bonus after each recharge, Unlimited
> 
> ![](https://assets.capsolver.com/prod/images/post/2024-03-29/fbc29472-886c-45b2-9eb2-2b307f6d9700.png)

Understanding these versions will help us navigate the challenges of reCAPTCHA effectively. Let’s dive into how we can use Puppeteer and CapSolver to handle these challenges and streamline our web scraping efforts.

### How CapSolver Can Help Solve reCAPTCHA

[CapSolver](https://www.capsolver.com/?utm_source=official&utm_medium=blog&utm_campaign=recaptchaphp) is a robust solution for handling CAPTCHA challenges, including reCAPTCHA. Here’s how you can integrate CapSolver into your workflow to simplify CAPTCHA resolution:

1. **Retrieve the Site Key**
   - Search the browser request logs for a request like `/recaptcha/api2/reload?k=6LcR_okUAAAAAPYrPe-HK_0RULO1aZM15ENyM-Mf`. The `k=` parameter is the site key you need.
   - If you provide an incorrect key, you’ll receive an error message like:
     ```
     Solve failed! response: {"errorId":1,"errorCode":"ERROR_INVALID_TASK_DATA","errorDescription":"Invalid site key","taskId":"1cd1e687-96dd-4f14-b8ef-18b5d144d9b8","status":"failed"}
     ```
   - If you use the wrong reCAPTCHA version (V2 or V3), or if there’s a mismatch between the target site type and the API request type, you might see:
     ```
     Solve failed! response: {"errorId":1,"errorCode":"ERROR_CAPTCHA_SOLVE_FAILED","errorDescription":"Failed to solve the captcha: 1001","taskId":"da450cbc-ff9d-439d-908a-77e7eb8852dd","status":"failed"}
     ```

2. **Set Up Your Environment**
   - Install the necessary packages:
     ```bash
     npm install axios puppeteer-core
     ```

3. **Write the Integration Code**


```php
<?php

require_once 'vendor/autoload.php';
use Nesk\Puphpeteer\Puppeteer;
use Nesk\Rialto\Data\JsFunction;
use GuzzleHttp\Client;

$puppeteer = new Puppeteer;
$browser = $puppeteer->launch();

// TODO: set your config
$api_key = "YOUR_API_KEY"; // your API key of CapSolver
$site_key = "6Le-wvkSAAAAAPBMRTvw0Q4Muexq9bi0DJwx_mJ-"; // site key of your target site
$site_url = "https://www.google.com/recaptcha/api2/demo"; // page URL of your target site

function capsolver()
{
    global $api_key, $site_key, $site_url;
    $client = new Client();
    $payload = [
        'clientKey' => $api_key,
        'task' => [
            'type' => 'ReCaptchaV2TaskProxyLess',
            'websiteKey' => $site_key,
            'websiteURL' => $site_url,
        ]
    ];

    try {
        $response = $client->post("https://api.capsolver.com/createTask", [
            'json' => $payload
        ]);
        $data = json_decode($response->getBody(), true);
        $task_id = $data['taskId'] ?? null;
        if (!$task_id) {
            echo "Failed to create task: " . json_encode($data) . PHP_EOL;
            return null;
        }
        echo "Got taskId: " . $task_id . PHP_EOL;

        while (true) {
            sleep(1);

            $getResultPayload = [
                'clientKey' => $api_key,
                'taskId' => $task_id
            ];
            $resp = $client->post("https://api.capsolver.com/getTaskResult", [
                'json' => $getResultPayload
            ]);
            $data = json_decode($resp->getBody(), true);
            $status = $data['status'] ?? null;

            if ($status === "ready") {
                return $data['solution']['gRecaptchaResponse'];
            }
            if ($status === "failed" || isset($data['errorId'])) {
                echo "Solve failed! response: " . json_encode($data) . PHP_EOL;
                return null;
            }
        }
    } catch (\Exception $e) {
        echo "Error: " . $e->getMessage() . PHP_EOL;
        return null;
    }
}

function reqSite()
{
    global $site_url, $browser;
    $token = capsolver();
    if ($token === null) {
        return;
    }
    echo $token . PHP_EOL;

    $page = $browser->newPage();
    $page->goto($site_url);
    $evaluate_script = <<<EOD
        document.getElementById("g-recaptcha-response").value="$token";
        onSuccess("$token");
    EOD;
    $page->evaluate(JsFunction::createWithBody($evaluate_script));
    $product_element = $page->querySelector('#recaptcha-demo-submit');
    if ($product_element instanceof ElementHandle) {
        $product_element->click();
    } else {
        echo 'Element not found.' . PHP_EOL;
    }
}

reqSite();
```


   

## Conclusion

By leveraging Puppeteer and CapSolver, you can effectively bypass reCAPTCHA challenges and streamline your web scraping tasks. CapSolver’s robust API handles CAPTCHA resolution with efficiency, while Puppeteer enables seamless automation of web interactions. Together, these tools provide a powerful solution for overcoming CAPTCHA barriers and ensuring smooth data extraction processes.

Whether you’re dealing with reCAPTCHA v2, v3, or Enterprise, integrating [CapSolver](https://www.capsolver.com/?utm_source=official&utm_medium=blog&utm_campaign=recaptchaphp) with Puppeteer can simplify your workflow and enhance your scraping efficiency. If you encounter any issues or need further assistance, both CapSolver and Puppeteer offer comprehensive documentation and support to help you navigate any challenges.
