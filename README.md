## Overview

This section will guide you in creating a framework for integrating Traknpay Payment Gateway with your android app. 

![Overview](https://traknpaypg.github.io/doc/images/overview.png?raw=true)

## Sample App
To understand the Traknpay payment flow, you can download our sample app [here](https://github.com/traknpaypg/traknpaypg.github.io).

### Prerequisites

1. You should be a registered and approved merchant with Traknpay. If not registered, please [register here!](https://biz.traknpay.in/auth/register)
2. You should have received the SALT and API key from Traknpay.

### Server Side Setup

To prevent the data tampering(and ensure data integrity) between the your app and Traknpay, you will need to setup up an API to calculate an encrypted value or checksum known as hash from the payment request parameters and SALT key before sending it to the 
Traknpay server.

```markdown
Traknpay uses **SHA512** cryptographic hash function to prevent data tampering. To calculate the hash, a secure 
private key known as **SALT key** will be provided by Traknpay that needs to be stored very **securely in your 
server**. Any compromise of the salt may lead to data tampering. 

# The hash generation code has 3 components:

1. Concatenate the request parameters(after trimming the blank spaces) separated by pipeline in the 
order below:   

`hash_data="SALT|address_line_1|address_line_2|amount|api_key|city|country|currency|description|email|hash
|mode|name|order_id|phone|return_url|state|udf1|udf2|udf3|udf4|udf5|zip_code"`

2. Change the string value obtained in step 1 to UPPERCASE.

3. Calculate the hash of the string value obtained in step 2 using "sha512" algorithm(all major languages 
would have an inhouse function to calculate the hash using SHA-512) and response the hash value to the 
android app.

```

Hash Generation of Payment Request for different languages has been given below:

```markdown
**Java Servlet Code**

`public class PaymentRequest extends HttpServlet {
	private static final long serialVersionUID = 1L;

	protected void doPost(HttpServletRequest request, HttpServletResponse response) 
	throws ServletException, IOException {
		// TODO Auto-generated method stub
		String salt = "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"; 
		
		String [] hash_columns = {"address_line_1", "address_line_2", "amount", "api_key", "city", 
		"country", "currency","description", "email", "mode", "name", "order_id", "phone", "return_url", 
		"state", "udf1", "udf2", "udf3", "udf4","udf5", "zip_code"};
		
		String hash_data = salt;
		
		for( int i = 0; i < hash_columns.length; i++)
		{
			if(request.getParameter(hash_columns[i]).length() > 0 ){
				hash_data += '|' + request.getParameter(hash_columns[i]);
			}    
			
		}
		
		String hash = "";
		try {
			 hash = getHashCodeFromString(hash_data);
		} catch (NoSuchAlgorithmException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
		JsonObject jsonResponse = new JsonObject();
                jsonResponse.addProperty("hash", hash);
             	jsonResponse.addProperty("status", "Kargopolov");
       		jsonResponse.addProperty("responseCode", "Kargopolov");


		response.setContentType("application/json");
		PrintWriter writer = response.getWriter();
		writer.print(jsonResponse);
        	writer.flush();

	}
	
	private static String getHashCodeFromString(String str) throws NoSuchAlgorithmException, 
	UnsupportedEncodingException {
			
		MessageDigest md = MessageDigest.getInstance("SHA-512");
	    	md.update(str.getBytes("UTF-8"));
	    	byte byteData[] = md.digest();

	    	//convert the byte to hex format method 1
	    	StringBuffer hashCodeBuffer = new StringBuffer();
	    	for (int i = 0; i < byteData.length; i++) {
	            hashCodeBuffer.append(Integer.toString((byteData[i] & 0xff) + 0x100, 16).substring(1));
	    	}
		return hashCodeBuffer.toString().toUpperCase();
	}
	
}`

```

```markdown
**PHP Code**

`try {
	$salt="XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX";

	$params["api_key"]=$_POST["api_key"];
	$params["amount"]=$_POST["amount"];
	$params["email"]=$_POST["email"];
	$params["name"]=$_POST["name"];
	$params["phone"]=$_POST["phone"];
	$params["order_id"]=$_POST["order_id"];
	$params["currency"]=$_POST["currency"];
	$params["description"]=$_POST["description"];
	$params["city"]=$_POST["city"];
	$params["state"]=$_POST["state"];
	$params["address_line_1"]=$_POST["address_line_1"];
	$params["address_line_2"]=$_POST["address_line_2"];
	$params["zip_code"]=$_POST["zip_code"];
	$params["country"]=$_POST["country"];
	$params["return_url"]=$_POST["return_url"];
	$params["mode"]=$_POST["mode"];
	$params["udf1"]=$_POST["udf1"];
	$params["udf2"]=$_POST["udf2"];
	$params["udf3"]=$_POST["udf3"];
	$params["udf4"]=$_POST["udf4"];
	$params["udf5"]=$_POST["udf5"];

	$hash_columns = [
		'name',
		'phone',
		'email',
		'description',
		'amount',
		'api_key',
		'order_id',
		'currency',
		'city',
		'state',
		'address_line_1',
		'address_line_2',
		'country',
		'zip_code',
		'return_url',
		'hash',
		'mode',
		'udf1',
		'udf2',
		'udf3',
		'udf4',
		'udf5'
	];

	sort($hash_columns);
	$hash_data = $salt;

	foreach ($hash_columns as $column) {
		if (isset($params[$column])) {
			if (strlen($params[$column]) > 0) {
				$hash_data .= '|' . $params[$column];
			}
		}
	}

	$hash = null;
	if (strlen($hash_data) > 0) {
		$hash = strtoupper(hash("sha512", $hash_data));
	}

	$output['hash'] = $hash;
	$output['status']=0;
	$output['responseCode']="Hash Created Successfully";

}catch(Exception $e) {
	$output['hash'] = "INVALID";
	$output['status']=1;
	$output['responseCode']=$e->getMessage();
}

echo json_encode($output);`

```

```markdown

{
    "amount": "2.00",
    "email": "test@gmail.com",
    "name": "Test Name",
    "phone": "9876543210",
    "order_id": "12",
    "currency": "INR",
    "description": "test",
    "city": "city",
    "state": "state",
    "zip_code": "123456",
    "country": "IND",
    "return_url": "https://ecaas.traknpay.in/response_page.php",
    "mode": "TEST",
    "udf1": "udf1",
    "udf2": "udf2",
    "udf3": "udf3",
    "udf4": "udf4",
    "udf5": "udf5",
    "address_line_1": "addl1",
    "address_line_2": "addl2",
    "api_key": "ce937655-4421-4c6b-b4fb-b57785ea55c4"
}

```
### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/traknpaypg/traknpaypg.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
