# cryptex

supplier -> manufacturer -> merchandiser -> brand -> customer

OCR for doc scanning & auto fillup forms (PO, INV)
Brands -> imagen for model marketing
Hedera + AI -> text & voice based product addition
Merchant -> Auto AI emailer handles client comms
IoT -> NFC tracking & scanning 
Maps -> open street maps for visibility of registered & connected supplier, manufacturer, buying house , brand

Primary Planning prompt for AI Agent(1200 tokens):
Cryptex: A modern AI & Hashgraph enabled ERP for textiles, this will be a textile supply chain management plaform with RBAC, you shall also help setup CI/CD via github actions & full deployment cycle,
The Goal:

Roles in the Web app : Supplier, Manufacturer, Merchandiser, Brand, Customer

User flows: Supplier get's order from Manufacturer in a PO sheet, they can directly upload the PO sheet to our platform, if it's sent via mail, we need an AI system to directly parse that PO & fill in necessary data fields , but I need you to actually run thorough research on how Textile suppliers actually work or is the work actually supposed to be handled by merchandiser, also we might need a custom form builder (we could use the industry standard open source tools) since different kinds of products may have different fields & requirements
Manufacturer,when he gets products from the supplier, he will first verify the info & send to the next step, will also similary handle PO orders via OCR-type service & form builder for custom form fields
Merchandiser, the AI agent will analyze & reply to email for the merchandiser & maintain communications, maybe we could add voice features too so it can make calls on behalf of the merchandiser, he will also verify the info received from manufacturer & send to brands
Brands, will use AI to analyze market trends & the AI will auto suggest what products they can launch, also analyze their incomes , how good they are doing by dealing with certain merhcandisers & manufacturers, but you will have to do extensive research on this as well
Finally Customer, the Customer will simply buy the products from a market place, verification/product life-cycle will be visible on the individual product page

Technical Choices: The entire system will be built upon the Hedera Hashgraph Ecosystem(https://docs.hedera.com/hedera/) with other components for specific features.
Requirements: All the data needs to be visible in HashScan (https://hashscan.io/testnet/home), basically when any user verifies any data, it will stored on hashscan(https://hedera.com/consensus-service), based on the number of verified data processed with need an algortihmic system to reward all users on the platform with digital coins (https://hedera.com/token-service), For the basic AI chatbot (i.e, answering simple questions about transactions on the platform we will use hedera's native AI agent system (https://www.npmjs.com/package/hedera-agent-kit#about-the-agent-kit-tools)
But there are other systems that will require more different AI approaches,
For document parsing & updating documents we will use a good document parsing library which will find the fields in the pdf , then send the parsed text to an llm using Requesty AI API(https://docs.requesty.ai/), the llm will use the given text & fields to understand which fields to fillup in the users form
We will also use requesty for Merhcnadiser side where it will read & reply on behalf of the user.
then for image generation, requesty models are capable of vision, but not sure if they support image generation, research on that
For market research we can just give the agent ability to surf the web or deepsearch type capabilities to draft a detailed Trend Plan for brands, so all these AI features will be avaiulable as separated options inside the same chat interface,
For basic voice feature, we can transcribe the voice using any package , which will just paste the text into chat bar , then user can send the text to ai
For Advanced Calling Agent , we can use LiveKit for voice agents
For frameworks & sdks for development
Hedera has both python & js/ts, since we are gonna be work8ing with AI, research what would be the better option
JS/TS: https://github.com/hiero-ledger/hiero-sdk-js
Python: https://github.com/hiero-ledger/hiero-sdk-python , https://github.com/hiero-ledger/hiero-did-sdk-python

For the backend, I am experienced in NestJS which comes in pre-built with a lot of stuff so we can just use a proper boiler plate repo & update it with stuff we need, but research on that as well that what would be the best option
For auth , also research what would be best (NestJS boilterplates often come pre-loaded with RBAC)
This is just the base idea, you will not code, but the final goal is to generate a detailed epscification I can hand off to a developer who can then begin implementaion.
So you will research, show me results & ask me & clarify questions to further research these topics, we may need to go through multiple iterations of deep search for this project

Bangladesh ,USA & EU for Now, Also I need a Tariff Agent which will gather latest info on Tariffs on the textile industry & help merchandiser or manufacturers take the right decisions, it will basically guide them
Basically Bangladeshi Supplier/Manufacturer/Merhchandiser & USA , EU based brands & customers, but research weather thias flow is actually accurate in a real world scenrio
General purpose (hence the custom fomr builder so that any manufacturer can add any product and also define their own schemas for what data should reside in the databse for their company, since there are many different types of products)
Research throughly on weather this is actually the industry standard or not
research about the entire supply chain
use context7