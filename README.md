[![Build status](https://ci.appveyor.com/api/projects/status/yt4ng6wtcd1nrd3b/branch/master?svg=true)](https://ci.appveyor.com/project/HristoKolev/tvdbsharper/branch/master) [![NuGet](https://img.shields.io/nuget/v/TvDbSharper.svg?maxAge=2592000?style=plastic)](https://www.nuget.org/packages/TvDbSharper/)

# How to installl
```PowerShell
Install-Package TvDbSharper
```
or with the NuGet UI in Visual Studio

# The client

There is one client you need to know about:

```C#
var client = new TvDbClient();
```
or

```C#
ITvDbClient client = new TvDbClient();
```

# Authentication

Before you do anything else you need to authenticate yourself.

* You will need an account on https://thetvdb.com/
* Then you will need to register an API key here: https://thetvdb.com/?tab=apiregister
* Then you can see all of the required information on your account page here: https://thetvdb.com/?tab=userinfo
* **Note**: The "Account Identifier" field holds the UserKey.

Then you can use the client like this:

```C#
await client.Authentication.AuthenticateAsync("ApiKey", "Username", "UserKey");
```

The session expires after 24 hours, but you can refresh that period by calling `RefreshTokenAsync`

```C#
await client.Authentication.RefreshTokenAsync();
```

# Series

Let's say that you need to get information about a specific series. Doctor Who. You can do it like this:

```C#
var response = await client.Series.GetAsync(78804);

Console.WriteLine(response.Data.SeriesName); //Doctor Who (2005)
Console.WriteLine(response.Data.Network); //BBC One
Console.WriteLine(response.Data.ImdbId); //tt0436992
```

If you want to get the episodes of a series... here the REST API shows its shortcomings. You can't do that on one line. You have to do multiple calls because it's paginated at 100 per page.

The code looks something like this:
```C#
const int SeriesId = 78804;

var tasks = new List<Task<TvDbResponse<BasicEpisode[]>>>();

var firstResponse = await client.Series.GetEpisodesAsync(SeriesId, 1);

for (int i = 2; i <= firstResponse.Links.Last; i++)
{
    tasks.Add(client.Series.GetEpisodesAsync(SeriesId, i));
}

// ReSharper disable once CoVariantArrayConversion
Task.WaitAll(tasks.ToArray());

var episodes = new List<BasicEpisode>(firstResponse.Data);

foreach (Task<TvDbResponse<BasicEpisode[]>> task in tasks)
{
    episodes.AddRange((await task).Data);
}

Console.WriteLine(episodes.Count); //263
```

To get all of the actors for a giver series, you can do this:
```C#
var response = await client.Series.GetActorsAsync(78804);

var theBestDoctor = response.Data.First();

Console.WriteLine(theBestDoctor.Name); //David Tennant
Console.WriteLine(theBestDoctor.Role); //10 (Tenth Doctor)
```
# Search

If you want to search for a series you have a few options. You can search by name, imdb ID or zap2it ID

Here is an example imdb search:

```C#
var response = await client.Search.SearchSeriesByImdbIdAsync("tt0436992");

var result = response.Data.First(); // We know there is only one

Console.WriteLine(result.SeriesName); //Doctor Who (2005)
Console.WriteLine(result.Network); //BBC One
Console.WriteLine(result.Id); //78804
```

and now that we know the series ID we can to other things with it.

# Updates

If you want to get all of the series that have been updated in a time interval you can do it like this:

```C#
var response = await client.Updates.GetAsync(new DateTime(2016, 10, 1), new DateTime(2016, 10, 5));

Update[] updates = response.Data;
```

If the interval is wider that 7 days it will be reduced to 7 days.

# Working with languages

You can get all available languages like that:

```C#
var response = await client.Languages.GetAllAsync();

Language[] languages = response.Data;
```

If you want to get details for a specific language, you can do it like that:

```C#
var response = await client.Languages.GetAsync(14);

Console.WriteLine(response.Data.EnglishName); //German
Console.WriteLine(response.Data.Abbreviation); //de
```

If you want to change the language that the client is working with, you can do it using the language abbreviation like this:

```C#
client.AcceptedLanguage = "de";
```
