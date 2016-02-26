# pylot
Pylot is a free open source tool for testing performance and scalability of web services. It runs HTTP load tests, which are useful for capacity planning, benchmarking, analysis, and system tuning.  
Pylot generates concurrent load (HTTP Requests), verifies server responses, and produces reports with metrics. Tests suites are executed and monitored from a GUI or shell/console.

Getting Started Guide
Platforms:

Console and Blocking Mode
Console and Blocking modes run on all platforms where Python 2.5+ can be installed. Tested on Windows XP, Vista, Ubuntu 8.04/8.10, Eee PC, Mac OS.

GUI Mode
Pylot GUI will run on all platforms that support Python and wxWidgets. The GUI has mostly been developed and tested on Windows, but looks decent on Linux and Mac. The application code is pure Python and uses a cross-platform toolkit.

Installing Pylot

Step 1: Download and unzip the latest Pylot release
Get the latest release here: Download Pylot

Step 2: Install Python 2.5+
Get installer from here: http://www.python.org/download

Step 3: Install wxPython (optional - used for GUI mode)
Get the installer from here: http://www.wxpython.org/download.php

Step 4: Install NumPy (optional - used for report graphs)
Get the installer from here: http://sourceforge.net/projects/numpy

Step 5: Install Matplotlib (optional - used for report graphs)
Get the installer from here: http://sourceforge.net/projects/matplotlib

Step 6: Run Pylot


GUI Mode:
> python run.py -g

Console and Blocking Mode - Command Line Options:
usage: run.py [options] args
  -a, --agents=NUM_AGENTS     :  number of agents
  -d, --duration=DURATION     :  test duration in seconds
  -r, --rampup=RAMPUP         :  rampup in seconds
  -i, --interval=INTERVAL     :  interval in milliseconds
  -x, --xmlfile=TEST_CASE_XML :  test case xml file
  -o, --output_dir=PATH       :  output directory
  -n, --name=TESTNAME         :  name of test
  -l, --log_msgs              :  log messages
  -b, --blocking              :  blocking mode
  -g, --gui                   :  start GUI
  -p, --port=PORT             :  xml-rpc listening port  

Starting Pylot Remotely:
Pylot contains an XML-RPC server that can be launched so you can start tests with a remote client.


Configuration Options:
The file /core/config.py contains some global configuration options. You can set certain defauls and alter certain behavior here. These options here are overridden if specified on the command line.


AGENTS = 1
DURATION = 60  # secs
RAMPUP = 0  # secs
INTERVAL = 0  # millisecs
TC_XML_FILENAME = 'testcases.xml'
OUTPUT_DIR = None
TEST_NAME = None
LOG_MSGS = False

GENERATE_RESULTS = True
SHUFFLE_TESTCASES = False  # randomize order of testcases per agent
WAITFOR_AGENT_FINISH = True  # wait for last requests to complete before stopping
SMOOTH_TP_GRAPH = 1  # secs.  smooth/dampen throughput graph based on an interval
SOCKET_TIMEOUT = 300  # secs
COOKIES_ENABLED = True

HTTP_DEBUG = False  # only useful when combined with blocking mode  
BLOCKING = False  # stdout blocked until test finishes, then result is returned as XML
GUI = False
Using Pylot

Step 1: Create Test Cases
Test cases are declared in an XML file named "testcases.xml", or a different XML file specified on the command line. This is the format that the test engine understands.

A test case is defined using the following syntax. Only the URL element is required.

<case>
  <url>URL</url>
  <method>HTTP METHOD</method>
  <body>REQUEST BODY CONTENT</body>
  <add_header>ADDITIONAL HTTP HEADER</add_header>
  <verify>STRING OR REGULAR EXPRESSION</verify>
  <verify_negative>STRING OR REGULAR EXPRESSION</verify_negative>
  <timer_group>TIMER GROUP NAME</timer_group>
</case>
Below is an example of the simplest possible test case file. It contains a single test case which will be executed continuously during the test run. The test case contains a URL for the service under test. Since no method or body defined, it will default to sending an HTTP GET to this resource. Since no verifications are defined, it will pass/fail the test case based on the HTTP status code it receives (pass if status is < 400).

<testcases>
  <case>
    <url>http://www.example.com/foo</url>
  </case>
</testcases>
We can add positive and negative verifications. A positive verification is a string or regular expression that must be contained in the response body. A negative verification is a string or regular expression that must not be contained in the response body.

<case>
    <url>http://www.goldb.org/foo</url>
    <verify>Copyright.*Corey Goldberg</verify>
    <verify_negative>Error</verify_negative>
<case>


Cookies:
Cookies are handled automatically. If a response is received with a "Set-cookie" header, the cookie will be set and passed back in the header of subsequent requests.



Example: Yahoo! Search Web Services (REST API)
Yahoo offers various REST Web Services to access search results. In this example, I will show how to create Pylot test cases to interact with the REST API.

Here is a simple GET request against the service:

http://search.yahooapis.com/WebSearchService/V1/webSearch?appid=YahooDemo&query=foo
A Pylot test case for this request would look like this:

<case>
  <url>http://search.yahooapis.com/WebSearchService/V1/webSearch?appid=YahooDemo&amp;query=foo</url>
</case>
Notice that the ampersand (&) in the URL was escaped with the code: "&amp;"
This is done becasue certain characters ("<" and "&") are illegal in XML documents. Since we are definig test cases within an XML doc, we must either escape these with ampersand codes, or place them within a CDATA section.

Yahoo also allows the query parameters to be passed in the POST data block. In this case we must also change the "Content-type" HTTP header to: "application/x-www-form-urlencoded". (Pylot defaults to "text/xml")

Here is a POST request against the service:

<case>
  <url>http://search.yahooapis.com/WebSearchService/V1/webSearch</url>
  <method>POST</method>
  <body><![CDATA[appid=YahooDemo&query=webinject]]></body>
  <add_header>Content-type: application/x-www-form-urlencoded</add_header>
</case>
Now that we know how to create individual cases, we can create a test case file containing several of these. In this example, our test case file contains Yahoo web search queries for: "foo", "bar", "baz"
<testcases>
  <case>
    <url>http://search.yahooapis.com/WebSearchService/V1/webSearch?appid=YahooDemo&amp;query=foo</url>
  </case>
    <case>
    <url>http://search.yahooapis.com/WebSearchService/V1/webSearch?appid=YahooDemo&amp;query=bar</url>
  </case>
    <case>
    <url>http://search.yahooapis.com/WebSearchService/V1/webSearch?appid=YahooDemo&amp;query=baz</url>
  </case>
</testcases>


Example: SOAP API
We can model our test cases to talk to any HTTP API. This example shows how you could send requests to a SOAP service. The SOAP envelope we need to send will be enclosed in the HTTP POST body.

<case>
  <url>http://www.example.org/StockPrice</url>
  <method>POST</method>
  <add_header>Content-Type: application/soap+xml; charset=utf-8</add_header>
  <body><!
    [CDATA[
    
      <!-- This is the SOAP Envelope  -->  
      <?xml version="1.0"?>
      <soap:Envelope xmlns:soap="http://www.w3.org/2001/12/soap-envelope"
        soap:encodingStyle="http://www.w3.org/2001/12/soap-encoding">
        <soap:Body xmlns:m="http://www.example.org/stock">
          <m:GetStockPrice>
            <m:StockName>IBM</m:StockName>
          </m:GetStockPrice>
        </soap:Body>
      </soap:Envelope>
      
    ]]>
  </body>
</case>


Example: Setting Static Variables/Parameters
You can define global parameters in your test case file. This is useful if you have a value shared among several test cases that you change often. In the example below, we define an "http_server" parameter and then use that token in a test case.

<testcases>
  <param name="http_server" value="http://www.example.com" />
  <case>
    <url>${http_server}/foo</url>
  </case>
</testcases>


Example: File-based HTTP Payloads
You may want to store POST data in an external file rather than declaring it directly in your testcase XML file. This is useful if you have very large POST BODYs or want to send binary data which can not be embedded in XML. Use the syntax below to pull data from a file and POST it at runtime.

<case>
  <url>http://www.example.com/foo</url>
  <method>POST</method>
  <body file="./myfile.dat"></body>
</case>


Step 2: Model Workload Scenario
Define a workload using the controls on the UI. Using the options below. you can create a steady-state or increasing load test.

Agents: number of agents (virtual users) to run
Rampup: time span over which agents are started. They will be evenly distributed throughout this time span. (see note below)
Interval: interval at which each user sends requests. The requests from each user agent are paced at even intervals (unless the respone time is slower thean the interval defined)
Duration: time span of the test


Step 3: Execute and Monitor

Run Modes
Console Mode: During the test, you can view real-time stats on the UI
Blocking Mode: STDOUT is blocked until test finishes, results are returned as XML
GUI Mode: Manage and view running tests with the GUI interface
At the end of a test run, an HTML report is automatically generated, showing test results and graphs.



Step 4: View Results
When a test is finished, a results directory is created and a report is automatically generated to summarize the test results. It includes various statistics and graphs for response times and throughput. A sample of the results report can be seen here:

Sample Report

Pylot also writes results to CSV text files so you can import them into your favorite spreadsheet to crunch numbers, generate statistics, and create graphs.
