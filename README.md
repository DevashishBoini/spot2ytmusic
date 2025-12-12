# Spotify to YouTube Music Migration Tool

A Jupyter notebook tool to migrate your Spotify playlists to YouTube Music with high-accuracy song matching.

## Features

- **Album-based matching**: Searches for albums first to find the exact track version
- **Fuzzy matching**: Handles slight differences in track names (brackets, parentheses, etc.)
- **Fallback searches**: If strict matching fails, tries progressively looser searches
- **Video fallback**: Falls back to YouTube videos when songs aren't available on YT Music
- **Artist-agnostic fallback**: Finds original versions when you have covers in your Spotify playlist
- **Rate limit handling**: Automatic retry with exponential backoff for API rate limits
- **Missing track logging**: Saves unfound tracks to `missing/` folder for manual review
- **Interactive UI**: Checkbox-based playlist selection in Jupyter

## Prerequisites

- Python 3.8+
- Jupyter Notebook or JupyterLab
- A Spotify account with playlists to migrate
- A YouTube Music account
- Spotify Developer credentials

## Setup

### 1. Clone the repository

```bash
git clone <your-repo-url>
cd spot2ytmusic
```

### 2. Install dependencies

The notebook will install dependencies automatically when you run it, but you can also install them manually:

```bash
pip install spotipy python-dotenv ytmusicapi ipywidgets
```

Or create a virtual environment first:

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install spotipy python-dotenv ytmusicapi ipywidgets
```

### 3. Set up Spotify credentials

1. Go to [Spotify Developer Dashboard](https://developer.spotify.com/dashboard)
2. Create a new app
3. Note down your **Client ID** and **Client Secret**
4. Add `http://127.0.0.1:8080/callback` as a Redirect URI in your app settings

Create a `.env` file in the project root:

```env
SPOTIFY_CLIENT_ID=your_client_id_here
SPOTIFY_CLIENT_SECRET=your_client_secret_here
```

### 4. Set up YouTube Music authentication

1. Open [YouTube Music](https://music.youtube.com) in Chrome
2. Make sure you're logged into your Google account
3. Open Developer Tools (F12) → **Network** tab
4. Reload the page
5. Click on any request to `music.youtube.com` like [browse, search, etc.]
6. Go to **Headers** tab → scroll to **Request Headers**
7. Copy ALL the request headers (from `:authority` to the last header)
8. Save them to a file called `raw_headers.txt` in the project folder

Example `raw_headers.txt` format:
```
:authority
music.youtube.com
:method
POST
:path
/youtubei/v1/browse?prettyPrint=false
...
cookie
YOUR_COOKIE_VALUE_HERE
...
```

## Usage

1. Open the notebook:
   ```bash
   jupyter notebook spot2ytmusic_script.ipynb
   ```

2. Run cells 0-8 to:
   - Install dependencies
   - Load Spotify credentials
   - Authenticate with Spotify (browser will open for OAuth)
   - Set up YouTube Music authentication

3. Run cell 9 to load the migration functions

4. Run cell 10 to:
   - Display your Spotify playlists with checkboxes
   - Select playlists to migrate
   - Click "Start migration" button

## How the matching works

The tool uses a multi-step matching algorithm:

1. **Album search**: Searches for the album on YT Music, then finds the exact track within it
2. **Song search with artist**: Searches `"{track} by {artist}"` with fuzzy title matching
3. **Video search with artist**: Falls back to YouTube videos if song not found
4. **Song search (track only)**: Ignores artist, searches by track name only
5. **Video search (track only)**: Last resort, finds any video matching the track name

This approach handles:
- Cover versions in your Spotify playlist → finds originals on YT Music
- Regional releases with different artist credits
- Slight naming differences between platforms

## Output

- **Created playlists**: New private playlists on your YouTube Music account
- **Missing tracks**: Saved to `missing/missing_{playlist_name}.txt` for manual review

## File Structure

```
spot2ytmusic/
├── spot2ytmusic_script.ipynb  # Main notebook
├── .env                        # Spotify credentials (create this)
├── raw_headers.txt             # YouTube Music headers (create this)
├── headers_auth.json           # Parsed YT headers (auto-generated)
├── missing/                    # Missing tracks logs (auto-created)
│   └── missing_*.txt
└── README.md
```

## Troubleshooting

### "Missing Spotify secrets" error
Make sure your `.env` file exists and contains valid `SPOTIFY_CLIENT_ID` and `SPOTIFY_CLIENT_SECRET`.

### YouTube Music authentication fails
- Headers expire after some time. Re-copy them from Chrome DevTools.
- Make sure you're logged into YouTube Music in Chrome.
- Ensure the `cookie` header is fully copied (it's very long).

### Rate limiting / 429 errors
The script handles this automatically with exponential backoff. Just wait for it to retry.

### Songs not found
Check the `missing/` folder for tracks that couldn't be matched. You can add them manually to your YT Music playlist.

## Dependencies

| Package | Purpose |
|---------|---------|
| `spotipy` | Spotify Web API client |
| `python-dotenv` | Load environment variables from `.env` |
| `ytmusicapi` | YouTube Music unofficial API client |
| `ipywidgets` | Interactive widgets for Jupyter |

## License

MIT License - feel free to use and modify as needed.



