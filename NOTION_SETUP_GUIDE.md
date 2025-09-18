# Notion Dynamic Multi-Agent System Setup Guide

## Overview

This guide will help you set up the converted Notion Dynamic Multi-Agent System that was originally built for HubSpot. The system uses Mistral AI reasoning combined with Notion databases to create an intelligent multi-agent system for business queries.

## What Was Changed

### 🔄 Key Conversions Made:

1. **Dependencies**: Replaced `hubspot-api-client` with `notion-client`
2. **Authentication**: Changed from HubSpot API key to Notion integration token
3. **Data Source**: Converted from HubSpot CRM objects to Notion databases
4. **API Connector**: Complete rewrite of `NotionConnector` class
5. **Data Handling**: Updated property parsing and transformation for Notion's format
6. **Rate Limiting**: Implemented Notion's 3 requests/second limit
7. **Batch Operations**: Adapted to Notion's sequential update pattern

## Prerequisites

- Python environment with Jupyter notebook support
- Notion workspace with admin access
- Mistral AI API key
- Basic understanding of Notion databases

## Step 1: Set Up Your Notion Workspace

### Create Notion Databases

You need to create three databases in your Notion workspace:

#### 1. Deals Database
```
Properties:
- Name (Title) - Deal name
- Amount (Number) - Deal value
- Stage (Select) - Options: Prospecting, Qualified, Proposal, Negotiation, Closed Won, Closed Lost
- Close Date (Date) - Expected or actual close date
- Priority (Select) - Options: High, Medium, Low
- Company (Relation) - Link to Companies database
- Contact (Relation) - Link to Contacts database
```

#### 2. Contacts Database
```
Properties:
- Name (Title) - Contact full name
- Email (Email) - Contact email address
- Phone (Phone) - Contact phone number
- Job Title (Rich Text) - Contact's job title
- Company (Relation) - Link to Companies database
- Stage (Select) - Options: Lead, Qualified, Customer, Lost
```

#### 3. Companies Database
```
Properties:
- Name (Title) - Company name
- Domain (URL) - Company website
- Industry (Select) - Options: Technology, Finance, Healthcare, Manufacturing, etc.
- Employee Count (Number) - Number of employees
- Annual Revenue (Number) - Company annual revenue
- Status (Select) - Options: Prospect, Customer, Partner
```

### Set Up Relations
- Link Deals → Companies (many-to-one)
- Link Deals → Contacts (many-to-one)
- Link Contacts → Companies (many-to-one)

## Step 2: Create Notion Integration

1. Go to https://www.notion.com/my-integrations
2. Click "New integration"
3. Give it a name like "AI Multi-Agent System"
4. Select your workspace
5. Click "Submit"
6. **Copy the Integration Token** - you'll need this

## Step 3: Share Databases with Integration

For each of your three databases:

1. Open the database page
2. Click the "..." menu in the top right
3. Go to "Connections"
4. Search for your integration name
5. Click "Connect"

## Step 4: Get Database IDs

For each database:

1. Open the database in Notion
2. Look at the URL: `https://www.notion.so/your-workspace/DATABASE_ID?v=...`
3. Copy the `DATABASE_ID` part (32-character string)

## Step 5: Configure the Notebook

Open the `notion_dynamic_multi_agent_system.ipynb` file and update these values:

```python
# Replace these with your actual values
NOTION_TOKEN = "secret_your_integration_token_here"
MISTRAL_API_KEY = "your_mistral_api_key_here"

# Your database IDs (32-character strings from URLs)
DEALS_DATABASE_ID = "your_deals_database_id"
CONTACTS_DATABASE_ID = "your_contacts_database_id" 
COMPANIES_DATABASE_ID = "your_companies_database_id"
```

## Step 6: Install Dependencies

Run this cell in your notebook:

```python
!pip install notion-client==2.2.1 mistralai==1.9.3
```

## Step 7: Test the System

### Basic Connection Test
```python
# Test database connection
orchestrator = AgentOrchestrator(
    notion_token=NOTION_TOKEN,
    mistral_api_key=MISTRAL_API_KEY,
    database_ids=DATABASE_IDS
)
```

### Sample Queries to Try

1. **Deal Priority Assignment**:
```python
query = "Assign priorities to all deals based on deal value."
result = orchestrator.process_query(query)
display(Markdown(result['final_answer']))
```

2. **Sales Analysis**:
```python
query = "Analyze our sales pipeline and identify bottlenecks in deal progression."
result = orchestrator.process_query(query)
```

3. **Market Intelligence**:
```python
query = """Analyze our current customer base to identify patterns in successful 
account profiles and recommend expansion opportunities."""
result = orchestrator.process_query(query)
```

## Key Differences from HubSpot Version

### Data Structure
- **HubSpot**: Objects with standard properties
- **Notion**: Pages with flexible property types

### API Patterns
- **HubSpot**: REST API with batch operations
- **Notion**: REST API with sequential operations + rate limiting

### Property Types
- **HubSpot**: Fixed property types per object
- **Notion**: Flexible property types per database

### Batch Updates
- **HubSpot**: Native batch API (100 records/request)
- **Notion**: Sequential updates with 0.4s delay between requests

## Performance Considerations

1. **Rate Limiting**: Notion allows ~3 requests/second
2. **Batch Operations**: Updates are processed sequentially
3. **Large Datasets**: Consider pagination for 100+ records
4. **Response Times**: Expect slower execution due to rate limits

## Troubleshooting

### Common Issues:

1. **Authentication Error**:
   - Verify integration token is correct
   - Ensure databases are shared with integration

2. **Property Not Found**:
   - Check property names match exactly
   - Verify property types are supported

3. **Rate Limit Exceeded**:
   - System includes automatic rate limiting
   - Reduce concurrent operations if needed

4. **Database Not Found**:
   - Verify database IDs are correct (32-character strings)
   - Ensure databases are accessible to integration

### Debug Mode

Add this for detailed logging:
```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

## Advanced Usage

### Custom Filters
```python
# Filter deals by stage
filters = {
    "property": "Stage",
    "select": {"equals": "Negotiation"}
}
deals = orchestrator.notion_connector.query_database(
    DEALS_DATABASE_ID, 
    filters=filters
)
```

### Custom Queries
```python
# Complex business intelligence query
query = """
Create a comprehensive customer segmentation analysis based on industry, 
deal size, and engagement patterns. Identify high-value segments and 
recommend targeted strategies for each segment.
"""
result = orchestrator.process_query(query)
```

## Support

If you encounter issues:

1. Check the Notion API documentation: https://developers.notion.com/
2. Verify your integration permissions
3. Test database access manually first
4. Check rate limiting if experiencing timeouts

## Next Steps

Once your system is working:

1. **Add Sample Data**: Populate your databases with test data
2. **Create Custom Queries**: Develop queries specific to your business needs  
3. **Extend Databases**: Add custom properties as needed
4. **Automate Workflows**: Schedule regular analysis runs
5. **Integrate with Other Tools**: Connect to your existing business stack

The converted system maintains all the sophisticated multi-agent reasoning capabilities of the original while leveraging Notion's flexible database structure for your business intelligence needs.