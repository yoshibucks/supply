<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8" />
        <meta name='viewport' content='width=device-width, user-scalable=no' />
        <title>Yoshi Bucks Supply</title>
Expand
yoshibucks.html
8 KB
MV — Today at 8:53 AM
Yes
mBlu — Today at 9:00 AM
🙏🏻
﻿
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8" />
        <meta name='viewport' content='width=device-width, user-scalable=no' />
        <title>Yoshi Bucks Supply</title>
    </head>
    <style>
      body { font-family: sans-serif; font-size: 14pt; text-align: center; }
      table { margin: auto; }
      th, td { text-align: right; padding: 0px 16px 0px 16px; }
      a { color: black; padding: 4px; }
      #links { padding-top: 32px; line-height: 1.5; }
      .bold { font-weight: bold; }
    </style>
    <body>
        <h1>Yoshi Bucks Supply</h1>
        <div id="results">Please wait</div>
        <div id="error"></div>
        <div id="links">
          <div class="bold">
            <a href="https://wax.atomichub.io/market?collection_name=yoshidrops&schema_name=yoshibucks">Get YoshiBucks</a>
            <a href="https://wax.alcor.exchange/swap?input=WAX-eosio.token&output=YOSHIBK-tokenizednft">Get Tokens</a>
          </div>
          <a href="https://yoshidrops.com/">Website</a>
          <a href="https://twitter.com/YoshiDrops">Twitter</a>
          <a href="https://discord.gg/JKPQFRBE">Discord</a>
        </div>
        <script>

collection = 'yoshidrops';
schema = 'yoshibucks';
token = 'YOSHIBK';
display = 'YB$ ';

endpoint = 'https://wax.greymass.com/';

calculate();

async function calculate()
{

  var results = '';

  var templates = await fetchtemplates();

  var burnFetches = [];
  for (var index = 0; index < templates.length; ++index)
  {
    burnFetches.push(fetchburncount(templates[index].templateId));
  }
  burnCounts = await Promise.all(burnFetches);
  for (var index = 0; index < templates.length; ++index)
  {
    templates[index].burnCount = burnCounts[index];
  }

  var tokenizednft = await fetchtokenizednft();

  var vaultFetches = [];
  for (var index = 0; index < templates.length; ++index)
  {
    vaultFetches.push(fetchvaultcount(tokenizednft[templates[index].templateId].Vault));
  }
  vaultCounts = await Promise.all(vaultFetches);
  for (var index = 0; index < templates.length; ++index)
  {
    templates[index].vaultCount = vaultCounts[index];
  }


  templates.sort((a, b) => { return a.denomination > b.denomination ? 1 : -1; });

  results += '<table>';
  results += '<tr><th>Denomination</th><th>Issued</th><th>Burned</th><th>Remaining</th><th>Tokenized</th><th>Tokenized Value</th><th>Total Value</th></tr>';

  var issued = 0;
  var burned = 0;
  var remaining = 0;
  var value = 0;
  var tokenized = 0;
  var tokenizedValue = 0;

  for (var index = 0; index < templates.length; ++index)
  {
    issued += templates[index].issueCount;
    burned += templates[index].burnCount;
    remaining += templates[index].issueCount - templates[index].burnCount;
    value += (templates[index].issueCount - templates[index].burnCount) * templates[index].denomination;
    tokenized += templates[index].vaultCount;
    tokenizedValue += templates[index].vaultCount * templates[index].denomination;

    results += '<tr>';
    results += '<td>' + display + comma(templates[index].denomination) + '</td>';
    results += '<td>' + comma(templates[index].issueCount) + '</td>';
    results += '<td>' + comma(templates[index].burnCount) + '</td>';
    results += '<td>' + comma(templates[index].issueCount - templates[index].burnCount) + '</td>';
    results += '<td>' + comma(templates[index].vaultCount) + '</td>';
    results += '<td>' + display + comma(templates[index].vaultCount * templates[index].denomination) + '</td>';
    results += '<td>' + display + comma((templates[index].issueCount - templates[index].burnCount) * templates[index].denomination) + '</td>';
    results += '</tr>';
  }

  results += '<tr><td>&nbsp;</td></tr><tr>';
  results += '<th>Totals</th>';
  results += '<td>' + comma(issued) + '</td>';
  results += '<td>' + comma(burned) + '</td>';
  results += '<td>' + comma(remaining) + '</td>';
  results += '<td>' + comma(tokenized) + '</td>';
  results += '<td>' + display + comma(tokenizedValue) + '</td>';
  results += '<td>' + display + comma(value) + '</td>';
  results += '</tr>';

  document.getElementById('results').innerHTML = results;
}

async function fetchtemplates()
{
  var results = [];
  var page = 1;
  var perPage = 100;

  while (true)
  {
    const url =
      "https://wax.api.atomicassets.io/atomicassets/v1/templates?" +
      "collection_name=" + collection +
      "&schema_name=" + schema +
      "&has_assets=true" +
      "&limit=" + perPage +
      "&page=" + page
    ;

    const response = await fetch(url, { headers: { "Content-Type": "text/plain" }, method: "GET" });
    const templateData = await response.json();

    if (templateData.success === true)
    {
      for (var templateIndex = 0; templateIndex < templateData.data.length; ++templateIndex)
      {
        results.push({
          templateId: templateData.data[templateIndex].template_id,
          denomination: parseInt(templateData.data[templateIndex].immutable_data.amount_spent),
          issueCount: parseInt(templateData.data[templateIndex].issued_supply),
        });
      }

      if (templateData.data.length < perPage)
      {
        break;
      }

      ++page;
    }
    else
    {
      document.getElementById('error').innerHTML = "ERROR: " + templateData.message;
      return [];
    }
  }

  return results;
}

async function fetchburncount(templateId)
{
  const url = 'https://wax.api.atomicassets.io/atomicassets/v1/templates/' + collection + '/' + templateId + '/stats';

  const response = await fetch(url, { headers: { "Content-Type": "text/plain" }, method: "GET" });
  const templateData = await response.json();

  if (templateData.success === true)
  {
    return parseInt(templateData.data.burned);
  }
  else
  {
    document.getElementById('error').innerHTML = "ERROR: " + templateData.message;
    return -1;
  }
}

async function fetchtokenizednft()
{
  var path = "v1/chain/get_table_rows";
  var data = JSON.stringify({json:true,code:"tokenizednft",scope:token,table:"values",limit:100});
  const response = await fetch(endpoint + path, { headers: { "Content-Type": "text/plain" }, body: data, method: "POST" });
  const body = await response.json();
  if (body.rows && Array.isArray(body.rows))
  {
    var results = {};

    for (var index = 0; index < body.rows.length; ++index)
    {
      results[body.rows[index].TemplateId] = body.rows[index];
    }
    return results;
  }
  else
  {
    document.getElementById('error').innerHTML = "ERROR: Unexpected response from endpoint";
    return {};
  }
}

async function fetchvaultcount(vault)
{
  const url = 'https://wax.api.atomicassets.io/atomicassets/v1/accounts/' + vault + '/' + collection;

  const response = await fetch(url, { headers: { "Content-Type": "text/plain" }, method: "GET" });
  const templateData = await response.json();

  if (templateData.success === true)
  {
    return templateData.data.schemas.length > 0 ? parseInt(templateData.data.schemas[0].assets) : 0;
  }
  else
  {
    document.getElementById('error').innerHTML = "ERROR: " + templateData.message;
    return -1;
  }
}

function comma(num)
{
  return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
}

        </script>
    </body>
</html>
yoshibucks.html
8 KB
