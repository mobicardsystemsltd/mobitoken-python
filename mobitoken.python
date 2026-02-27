import json
import base64
import hmac
import hashlib
import random
import requests

class MobicardTokenization:
    def __init__(self, merchant_id, api_key, secret_key):
        self.mobicard_version = "2.0"
        self.mobicard_mode = "LIVE"
        self.mobicard_merchant_id = merchant_id
        self.mobicard_api_key = api_key
        self.mobicard_secret_key = secret_key
        self.mobicard_service_id = "20000"
        self.mobicard_service_type = "TOKENIZATION"
        
        self.mobicard_token_id = str(random.randint(1000000, 1000000000))
        self.mobicard_txn_reference = str(random.randint(1000000, 1000000000))
    
    def tokenize_card(self, card_number, expiry_month, expiry_year, 
                     single_use_token=False, custom_data=None):
        """Tokenize card details"""
        
        mobicard_single_use_token_flag = "1" if single_use_token else "0"
        
        # Default custom data
        if custom_data is None:
            custom_data = {
                "mobicard_custom_one": "mobicard_custom_one",
                "mobicard_custom_two": "mobicard_custom_two",
                "mobicard_custom_three": "mobicard_custom_three",
                "mobicard_extra_data": "your_custom_data_here_will_be_returned_as_is"
            }
        
        # Create JWT Header
        jwt_header = {"typ": "JWT", "alg": "HS256"}
        encoded_header = base64.urlsafe_b64encode(
            json.dumps(jwt_header).encode()
        ).decode().rstrip('=')
        
        # Create JWT Payload
        jwt_payload = {
            "mobicard_version": self.mobicard_version,
            "mobicard_mode": self.mobicard_mode,
            "mobicard_merchant_id": self.mobicard_merchant_id,
            "mobicard_api_key": self.mobicard_api_key,
            "mobicard_service_id": self.mobicard_service_id,
            "mobicard_service_type": self.mobicard_service_type,
            "mobicard_token_id": self.mobicard_token_id,
            "mobicard_txn_reference": self.mobicard_txn_reference,
            "mobicard_single_use_token_flag": mobicard_single_use_token_flag,
            "mobicard_card_number": card_number,
            "mobicard_card_expiry_month": expiry_month,
            "mobicard_card_expiry_year": expiry_year,
            "mobicard_custom_one": custom_data.get("mobicard_custom_one", ""),
            "mobicard_custom_two": custom_data.get("mobicard_custom_two", ""),
            "mobicard_custom_three": custom_data.get("mobicard_custom_three", ""),
            "mobicard_extra_data": custom_data.get("mobicard_extra_data", "")
        }
        
        encoded_payload = base64.urlsafe_b64encode(
            json.dumps(jwt_payload).encode()
        ).decode().rstrip('=')
        
        # Generate Signature
        header_payload = f"{encoded_header}.{encoded_payload}"
        signature = hmac.new(
            self.mobicard_secret_key.encode(),
            header_payload.encode(),
            hashlib.sha256
        ).digest()
        encoded_signature = base64.urlsafe_b64encode(signature).decode().rstrip('=')
        
        jwt_token = f"{encoded_header}.{encoded_payload}.{encoded_signature}"
        
        # Make API Call
        url = "https://mobicardsystems.com/api/v1/card_tokenization"
        payload = {"mobicard_auth_jwt": jwt_token}
        
        try:
            response = requests.post(url, json=payload, verify=False, timeout=30)
            response_data = response.json()
            
            if response_data.get('status') == 'SUCCESS':
                return {
                    'status': 'SUCCESS',
                    'card_token': response_data['card_information']['card_token'],
                    'card_number_masked': response_data['card_information']['card_number_masked'],
                    'raw_response': response_data
                }
            else:
                return {
                    'status': 'ERROR',
                    'status_code': response_data.get('status_code'),
                    'status_message': response_data.get('status_message')
                }
                
        except Exception as e:
            return {'status': 'ERROR', 'error_message': str(e)}

# Usage
tokenizer = MobicardTokenization(
    merchant_id="4",
    api_key="YmJkOGY0OTZhMTU2ZjVjYTIyYzFhZGQyOWRiMmZjMmE2ZWU3NGIxZWM3ZTBiZSJ9",
    secret_key="NjIwYzEyMDRjNjNjMTdkZTZkMjZhOWNiYjIxNzI2NDQwYzVmNWNiMzRhMzBjYSJ9"
)

result = tokenizer.tokenize_card(
    card_number="4242424242424242",
    expiry_month="02",
    expiry_year="28",
    single_use_token=False
)

if result['status'] == 'SUCCESS':
    print(f"Token: {result['card_token']}")
    print(f"Masked Card: {result['card_number_masked']}")
    print("Store token and masked card in your database.")
else:
    print(f"Error: {result.get('status_message')}")
