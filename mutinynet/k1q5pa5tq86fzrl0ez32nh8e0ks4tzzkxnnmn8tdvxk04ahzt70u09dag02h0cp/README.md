
To use these files:


Test "Create":

- (key-based) using  "keys.json"(not in mutinynet yet) -> btc1::Create() -> "initialDidDoc.json"
- (external-based) using "initialDidDoc.json" -> convert to intermediate -> btc1::Create() -> "initialDidDoc.json" // todo: need new "x1" directory + "intermediateDidDoc.json"

Test "Read":
- using "did.txt" + "resolutionOptions.json" -> btc1::Read() -> "targetDocument.json"


Test "Update": 
- using initial "initialDidDoc.json" + "resolutionOptions.json" -> btc1::Update() -> "targetDocument.json" 


