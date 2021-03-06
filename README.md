# TON Client

## Overview
This is a java wrapper for telegram open network client. Base on original [native-lib.cpp](https://github.com/ton-blockchain/ton/blob/master/example/android/native-lib.cpp).

## Supported OS

* <b>Linux</b> - build on Ubuntu 18.04 LTS and Java 8.

* <b>MacOS</b> - build on Sierra and Java 8.

Supported features
-----

* Keys, create, import and export as seed phrase.

* Accounts, create wallet, send grams, get statuses.

* Transactions, account transaction history, messages and binary data.

Example docker and http api
----
You can find docker example and simple http api [here](https://github.com/broxus/ton-api).

Build native-lib
----
You can use [docker](docker/Dockerfile) container to build native-lib from sources.
```bash
# run this commands in repo root

# prepare builder container with all needed dependencies
docker build --tag ton-builder -f docker/Dockerfile .

# now run build process where:
#  GIT_URL is your ton repository
#  COMMIT_SHA (defaulat master HEAD) sha1 of commit to build
#  BUILD_THREAD_COUNT (default is 1) is number of threads used for building
docker run -ti --rm --mount type=bind,source="$(pwd)/build",target=/workdir/build ton-builder url=GIT_URL commit=COMMIT_SHA threads=BUILD_THREAD_COUNT

# copy native lib and updated src
cp build/ton/build/native-lib/libnative-lib.so src/main/resources/nativelib/libnative-lib.so
cp build/ton/native-lib/src/com/broxus/ton/TonApi.java src/main/java/com/broxus/ton/TonApi.java
```

Example
----
```java
import io.broxus.ton.TonClient;
import io.broxus.ton.TonApi;

class ExampleApp {
 
    private TonClient client = TonClient.create(
                updatesHandler, 
                updatesExceptionHandler, 
                defaultExceptionHandler
            );
    
    // lite client config https://test.ton.org/ton-lite-client-test1.config.json
    private final String config = "{ \"liteservers\": [ { \"ip\": 1137658550, \"port\": 4924, \"id\": { \"@type\": \"pub.ed25519\", \"key\": \"peJTw/arlRfssgTuf9BMypJzqOi7SXEqSPSWiEw2U1M=\" } } ], \"validator\": { \"@type\": \"validator.config.global\", \"zero_state\": { \"workchain\": -1, \"shard\": -9223372036854775808, \"seqno\": 0, \"root_hash\": \"VCSXxDHhTALFxReyTZRd8E4Ya3ySOmpOWAS4rBX9XBY=\", \"file_hash\": \"eh9yveSz1qMdJ7mOsO+I+H77jkLr9NpAuEkoJuseXBo=\" } } }";
    
    // path to directory with state (keystore and last block)
    private final String keystore = ".";
    
    public static void main(args String[]) {
        
        // init ton client first
        client.send(new TonApi.Init(
            new TonApi.Options(
                new TonApi.Config(
                    config, // lite client config
                    "", // chain name
                    false, // network callbacks
                    false // cache usage
                ),                
                new TonApi.KeyStoreTypeDirectory(keystore) // or new TonApi.KeyStoreTypeInMemory() 
            )
        ), new TonClient.ResultHandler() {
            
            @Override
            void onResult(TonApi.Object object) {
                System.out.println("Inited!");
                
                // request giver state
                client.send(new TonApi.TestGiverGetAccountState(), new TonClient.ResultHandler() {
                    
                    @Override
                    void onResult(TonApi.Object object) {

                        if(object instanceof TonApi.Error) {
                            
                            TonApi.Error error = (TonApi.Error) object;
                            System.err.println("Get an error on test giver request state " + error.message);
                            
                        } else if (object instanceof TonApi.TestGiverAccountState) {

                            TonApi.TestGiverAccountState giverState = (TonApi.TestGiverAccountState) object;
                            System.out.println("Giver seq " + giverState.seqno);
                            System.out.println("Giver balance " + giverState.balance);

                            // send gram request to giver
                            client.send(new TonApi.TestGiverSendGrams(
                                new TonApi.AccountAddress(
                                    // your account address
                                ),
                                giverState.seqno,
                                5L * 1000000000L, // 5 grams
                                null
                            ));
                            
                        } else {
                            
                            // ...
                        }
                    }
                });
            }
        });
        
        // ... 
        
        // at this moment never close client
        // client.close();
    }
    
    
}
```

## License

```
Copyright 2019 FINEX FUTURE LTD

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at [apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
