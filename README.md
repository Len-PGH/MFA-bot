# MFA-bot
Send and verify mfa with Signalwire AI

Uses Functions `send_mfa` and `verify_mfa`.


send_mfa
----------

### Perl Script Explanation

This Perl script is designed to handle an HTTP request, process data, and interact with the SignalWire REST API for sending SMS. Below is a breakdown of its main components:

### Request Handling and Data Processing

1. **Initialize Environment and Request Objects:**
    ```perl
    my $env = shift;
    my $req = Plack::Request->new($env);
    ```
    - The script begins by retrieving the environment (`$env`) and creating a new `Plack::Request` object.

2. **SignalWire Markup Language (ML) and JSON Parsing:**
    ```perl
    my $swml = SignalWire::ML->new;
    my $post_data = decode_json($req->raw_body);
    ```
    - A new `SignalWire::ML` object is instantiated.
    - The request's raw body is parsed as JSON.

3. **Extracting Data from the Request:**
    ```perl
    my $data = $post_data->{argument}->{parsed}->[0];
    my $agent = $req->param('agent_id');
    ```
    - Extracts specific data from the parsed JSON.
    - Retrieves the `agent_id` parameter from the request.

4. **Access Caller ID Number:**
    ```perl
    my $cid = $post_data->{'caller_id_num'};
    ```
    - Retrieves the caller ID number from the post data.

### SignalWire REST API Interaction

1. **Initialize the SignalWire REST API:**
    ```perl
    my $sw = SignalWire::RestAPI->new(
        AccountSid  => 'effa3364-a8b1-461c-9512-0545bce90b65',
        AuthToken   => 'PT.......',
        Space       => 'replace me with the space name',
        API_VERSION => 'api/relay/rest'
    );
    ```
    - Initializes the SignalWire REST API client with necessary credentials.

2. **Send SMS via SignalWire API:**
    ```perl
    my $response = $sw->POST("mfa/sms", {
        to           => $cid,
        from         => '+15551234567',
        message      => 'Your number to verify is: ',
        allow_alpha  => 'true',
        token_length => 6,
        valid_for    => 7200,
        max_attempts => 4
    });
    ```
    - Sends an SMS to the caller ID number with specific parameters.

### Response Handling

1. **Decode the SMS Response:**
    ```perl
    my $decoded_sms_response = $json->decode($response->{content});
    ```
    - Decodes the content of the SMS response.

2. **Generate a Response:**
    - Depending on the success of the SMS sending operation, the script generates an appropriate response using SignalWire ML.

3. **Finalize Response:**
    ```perl
    $res->content_type('application/json');
    return $res->finalize;
    ```
    - Sets the content type of the response to `application/json`.
    - Finalizes and returns the response.

### Broadcasting

Throughout the script, there are calls to `broadcast_by_agent_id` function, which seem to be broadcasting certain events or data to an agent. This functionality is commented out in some parts of the script.

---

verify_mfa
-----------

### Perl Script Overview

This Perl script is designed for handling HTTP requests and interacting with the SignalWire REST API for Multi-Factor Authentication (MFA) verification. It processes incoming data, verifies it using the SignalWire API, and returns a response based on the verification outcome.

## Script Breakdown

### Initialization and Request Handling

- **Environment and Request Initialization:**
    ```perl
    my $env = shift;
    my $req = Plack::Request->new($env);
    ```
    Initializes the environment and creates a new Plack request object.

- **SignalWire Markup Language (ML) and JSON Parsing:**
    ```perl
    my $swml = SignalWire::ML->new;
    my $post_data = decode_json($req->raw_body);
    ```
    Creates a new `SignalWire::ML` object and parses the request's raw body as JSON.

- **Extracting Data from Request:**
    ```perl
    my $data = $post_data->{argument}->{parsed}->[0];
    my $agent = $req->param('agent_id');
    my $mfa = $post_data->{meta_data}->{mfa};
    ```
    Extracts specific data, including `agent_id` and MFA metadata, from the parsed JSON.

### SignalWire REST API Interaction

- **Initialize the SignalWire REST API:**
    ```perl
    my $sw = SignalWire::RestAPI->new(
        AccountSid  => 'fd0a3364-a8b1-461c-9512-0545bce90b65',
        AuthToken   => 'PT3769437b4bff6eedab374a13c46f352447d7bafd89fbf16a',
        Space       => 'len',
        API_VERSION => 'api/relay/rest'
    );
    ```
    Initializes the SignalWire REST API client with the provided credentials.

- **MFA Verification Request:**
    ```perl
    my $verify = $sw->POST("mfa/$mfa->{id}/verify", token => $data->{token});
    ```
    Sends a POST request to the SignalWire API for MFA verification using the token from the request data.

### Response Handling

- **Decode the Verification Response:**
    ```perl
    my $resp = decode_json($verify->{content});
    ```
    Decodes the content of the verification response.

- **Generate and Finalize Response:**
    - Depending on the `success` field of the response, the script generates an appropriate JSON response.
    - If successful, it indicates verification success; otherwise, it prompts to try again.

    ```perl
    $res->content_type('application/json');
    return $res->finalize;
    ```
    Sets the content type of the response to `application/json` and finalizes it.

### Broadcasting

- **Broadcasting Events:**
    ```perl
    broadcast_by_agent_id($agent, $data);
    broadcast_by_agent_id($agent, $verify);
    ```
    The script contains calls to `broadcast_by_agent_id` function, which broadcasts specific events or data to an agent. This functionality is used for both the initial data and the verification result.

## Additional Notes

- The script uses `Plack::Request` and `Plack::Response` for handling HTTP requests and responses, typical in a PSGI/Plack web application context.
- Sensitive data like authentication tokens and account IDs are included in the script and should be handled securely in a production environment.


Note: Remember to handle sensitive data like authentication tokens securely and replace placeholders with actual values in production.



---------------------

### SignalWire

#### SignalWire’s AI Agent for Voice allows you to build and deploy your own digital employee. Powered by advanced natural language processing (NLP) capabilities, your digital employee will understand caller intent, retain context, and generally behave in a way that feels “human-like”.  In fact, you may find that it behaves exactly like your best employee, elevating the customer experience through efficient, intelligent, and personalized interactions.
