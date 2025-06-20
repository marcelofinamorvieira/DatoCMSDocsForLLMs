# Icon Type Reference

The Icon type in DatoCMS Plugin SDK provides a flexible way to specify icons throughout your plugin. Icons can be either Font Awesome identifiers or custom SVG definitions.

## Icon Type Definition

```typescript
type Icon = AwesomeFontIconIdentifier | SvgIcon;

type AwesomeFontIconIdentifier = string; // One of 4,205 Font Awesome icons

interface SvgIcon {
  type: 'svg';
  viewBox: string;
  content: string;
}
```

## Usage Examples

### Using Font Awesome Icons

```typescript
import type { Icon } from 'datocms-plugin-sdk';

// Simple string identifier
const icon1: Icon = 'user';
const icon2: Icon = 'cog';
const icon3: Icon = 'plus-circle';

// In a dropdown action
const action = {
  id: 'my-action',
  label: 'My Action',
  icon: 'rocket' // Font Awesome icon
};
```

### Using Custom SVG Icons

```typescript
const customIcon: Icon = {
  type: 'svg',
  viewBox: '0 0 24 24',
  content: '<path d="M12 2L2 7v10c0 5.55 3.84 10.74 9 12 5.16-1.26 9-6.45 9-12V7l-10-5z"/>'
};

// In a plugin configuration
const myPlugin = {
  mainNavigationTabs() {
    return [{
      id: 'custom-tab',
      label: 'Custom Tab',
      icon: customIcon
    }];
  }
};
```

## Font Awesome Icon Categories

The SDK includes 4,205 Font Awesome icons. Here are the main categories:

### Common UI Icons
- `plus`, `minus`, `check`, `times`, `close`
- `edit`, `pen`, `pencil`, `trash`, `delete`
- `save`, `download`, `upload`, `sync`, `refresh`
- `search`, `filter`, `sort`, `list`, `grid`
- `cog`, `cogs`, `wrench`, `tools`, `settings`

### Arrows & Navigation
- `arrow-up`, `arrow-down`, `arrow-left`, `arrow-right`
- `chevron-up`, `chevron-down`, `chevron-left`, `chevron-right`
- `angle-up`, `angle-down`, `angle-left`, `angle-right`
- `caret-up`, `caret-down`, `caret-left`, `caret-right`
- `long-arrow-alt-up`, `long-arrow-alt-down`, `long-arrow-alt-left`, `long-arrow-alt-right`

### Status & Alerts
- `info`, `info-circle`, `question`, `question-circle`
- `exclamation`, `exclamation-circle`, `exclamation-triangle`
- `check-circle`, `times-circle`, `ban`, `warning`
- `bell`, `bell-slash`, `notification`, `alert`

### File & Document
- `file`, `file-alt`, `file-text`, `file-code`
- `file-pdf`, `file-word`, `file-excel`, `file-powerpoint`
- `file-image`, `file-video`, `file-audio`, `file-archive`
- `folder`, `folder-open`, `folder-plus`, `folder-minus`

### Media & Content
- `image`, `images`, `camera`, `video`, `film`
- `music`, `volume-up`, `volume-down`, `volume-mute`
- `play`, `pause`, `stop`, `forward`, `backward`
- `microphone`, `headphones`, `podcast`, `broadcast`

### Social & Communication
- `envelope`, `envelope-open`, `at`, `inbox`
- `comment`, `comments`, `chat`, `message`
- `share`, `share-alt`, `retweet`, `link`
- `facebook`, `twitter`, `instagram`, `linkedin`

### Complete Icon List

<details>
<summary>Click to expand the complete list of all 4,205 Font Awesome icons</summary>

#### Numbers & Letters
`0`, `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9`, `a`, `b`, `c`, `d`, `e`, `f`, `g`, `h`, `i`, `j`, `k`, `l`, `m`, `n`, `o`, `p`, `q`, `r`, `s`, `t`, `u`, `v`, `w`, `x`, `y`, `z`, `h1`, `h2`, `h3`, `h4`, `h5`, `h6`

#### Full Alphabetical List
`abacus`, `accent`, `access`, `accessible-icon`, `accusoft`, `acorn`, `ad`, `add`, `address-book`, `address-card`, `adjust`, `adn`, `adobe`, `adversal`, `affiliatetheme`, `air-conditioner`, `air-freshener`, `airbnb`, `alarm-clock`, `alarm-exclamation`, `alarm-plus`, `alarm-snooze`, `album`, `album-collection`, `algolia`, `alien`, `alien-monster`, `align-center`, `align-justify`, `align-left`, `align-right`, `align-slash`, `alipay`, `allergies`, `amazon`, `amazon-pay`, `ambulance`, `american-sign-language-interpreting`, `amilia`, `amp-guitar`, `analytics`, `anchor`, `android`, `angel`, `angellist`, `angle-double-down`, `angle-double-left`, `angle-double-right`, `angle-double-up`, `angle-down`, `angle-left`, `angle-right`, `angle-up`, `angry`, `angrycreative`, `angular`, `ankh`, `app-store`, `app-store-ios`, `apper`, `apple`, `apple-alt`, `apple-crate`, `apple-pay`, `archive`, `archway`, `arrow-alt-circle-down`, `arrow-alt-circle-left`, `arrow-alt-circle-right`, `arrow-alt-circle-up`, `arrow-alt-down`, `arrow-alt-from-bottom`, `arrow-alt-from-left`, `arrow-alt-from-right`, `arrow-alt-from-top`, `arrow-alt-left`, `arrow-alt-right`, `arrow-alt-square-down`, `arrow-alt-square-left`, `arrow-alt-square-right`, `arrow-alt-square-up`, `arrow-alt-to-bottom`, `arrow-alt-to-left`, `arrow-alt-to-right`, `arrow-alt-to-top`, `arrow-alt-up`, `arrow-circle-down`, `arrow-circle-left`, `arrow-circle-right`, `arrow-circle-up`, `arrow-down`, `arrow-from-bottom`, `arrow-from-left`, `arrow-from-right`, `arrow-from-top`, `arrow-left`, `arrow-right`, `arrow-square-down`, `arrow-square-left`, `arrow-square-right`, `arrow-square-up`, `arrow-to-bottom`, `arrow-to-left`, `arrow-to-right`, `arrow-to-top`, `arrow-up`, `arrows`, `arrows-alt`, `arrows-alt-h`, `arrows-alt-v`, `arrows-h`, `arrows-v`, `artstation`, `assistive-listening-systems`, `asterisk`, `asymmetrik`, `at`, `atlas`, `atlassian`, `atom`, `atom-alt`, `audible`, `audio-description`, `autoprefixer`, `avianex`, `aviato`, `award`, `aws`, `axe`, `axe-battle`, `baby`, `baby-carriage`, `backpack`, `backspace`, `backward`, `bacon`, `bacteria`, `bacterium`, `badge`, `badge-check`, `badge-dollar`, `badge-percent`, `badge-sheriff`, `badger-honey`, `bags-shopping`, `bahai`, `balance-scale`, `balance-scale-left`, `balance-scale-right`, `ball-pile`, `ballot`, `ballot-check`, `ban`, `band-aid`, `bandcamp`, `banjo`, `barcode`, `barcode-alt`, `barcode-read`, `barcode-scan`, `bars`, `baseball`, `baseball-ball`, `basketball-ball`, `basketball-hoop`, `bat`, `bath`, `battery-bolt`, `battery-empty`, `battery-full`, `battery-half`, `battery-quarter`, `battery-slash`, `battery-three-quarters`, `battle-net`, `bed`, `bed-alt`, `bed-bunk`, `bed-empty`, `beer`, `behance`, `behance-square`, `bell`, `bell-exclamation`, `bell-on`, `bell-plus`, `bell-school`, `bell-school-slash`, `bell-slash`, `bells`, `betamax`, `bezier-curve`, `bible`, `bicycle`, `bike-messenger`, `biking`, `biking-mountain`, `bimobject`, `binoculars`, `biohazard`, `birthday-cake`, `bitbucket`, `bitcoin`, `bity`, `black-tie`, `blackberry`, `blanket`, `blender`, `blender-phone`, `blind`, `blinds`, `blinds-open`, `blinds-raised`, `blog`, `blogger`, `blogger-b`, `bluetooth`, `bluetooth-b`, `bold`, `bolt`, `bomb`, `bone`, `bone-break`, `bong`, `book`, `book-alt`, `book-dead`, `book-heart`, `book-medical`, `book-open`, `book-reader`, `book-spells`, `book-user`, `bookmark`, `books`, `books-medical`, `boombox`, `boot`, `booth-curtain`, `bootstrap`, `border-all`, `border-bottom`, `border-center-h`, `border-center-v`, `border-inner`, `border-left`, `border-none`, `border-outer`, `border-right`, `border-style`, `border-style-alt`, `border-top`, `bow-arrow`, `bowling-ball`, `bowling-pins`, `box`, `box-alt`, `box-ballot`, `box-check`, `box-fragile`, `box-full`, `box-heart`, `box-open`, `box-tissue`, `box-up`, `box-usd`, `boxes`, `boxes-alt`, `boxing-glove`, `brackets`, `brackets-curly`, `braille`, `brain`, `bread-loaf`, `bread-slice`, `briefcase`, `briefcase-medical`, `bring-forward`, `bring-front`, `broadcast-tower`, `broom`, `browser`, `brush`, `btc`, `buffer`, `bug`, `building`, `bullhorn`, `bullseye`, `bullseye-arrow`, `bullseye-pointer`, `burger-soda`, `burn`, `buromobelexperte`, `burrito`, `bus`, `bus-alt`, `bus-school`, `business-time`, `buy-n-large`, `buysellads`, `cabinet-filing`, `cactus`, `calculator`, `calculator-alt`, `calendar`, `calendar-alt`, `calendar-check`, `calendar-day`, `calendar-edit`, `calendar-exclamation`, `calendar-minus`, `calendar-plus`, `calendar-star`, `calendar-times`, `calendar-week`, `camcorder`, `camera`, `camera-alt`, `camera-home`, `camera-movie`, `camera-polaroid`, `camera-retro`, `campfire`, `campground`, `canadian-maple-leaf`, `candle-holder`, `candy-cane`, `candy-corn`, `cannabis`, `capsules`, `car`, `car-alt`, `car-battery`, `car-building`, `car-bump`, `car-bus`, `car-crash`, `car-garage`, `car-mechanic`, `car-side`, `car-tilt`, `car-wash`, `caravan`, `caravan-alt`, `caret-circle-down`, `caret-circle-left`, `caret-circle-right`, `caret-circle-up`, `caret-down`, `caret-left`, `caret-right`, `caret-square-down`, `caret-square-left`, `caret-square-right`, `caret-square-up`, `caret-up`, `carrot`, `cars`, `cart-arrow-down`, `cart-plus`, `cash-register`, `cassette-tape`, `cat`, `cat-space`, `catalogs`, `cauldron`, `cc-amazon-pay`, `cc-amex`, `cc-apple-pay`, `cc-diners-club`, `cc-discover`, `cc-jcb`, `cc-mastercard`, `cc-paypal`, `cc-stripe`, `cc-visa`, `cctv`, `centercode`, `centos`, `certificate`, `chair`, `chair-office`, `chalkboard`, `chalkboard-teacher`, `charging-station`, `chart-area`, `chart-bar`, `chart-line`, `chart-line-down`, `chart-network`, `chart-pie`, `chart-pie-alt`, `chart-scatter`, `check`, `check-circle`, `check-double`, `check-square`, `cheese`, `cheese-swiss`, `cheeseburger`, `chess`, `chess-bishop`, `chess-bishop-alt`, `chess-board`, `chess-clock`, `chess-clock-alt`, `chess-king`, `chess-king-alt`, `chess-knight`, `chess-knight-alt`, `chess-pawn`, `chess-pawn-alt`, `chess-queen`, `chess-queen-alt`, `chess-rook`, `chess-rook-alt`, `chevron-circle-down`, `chevron-circle-left`, `chevron-circle-right`, `chevron-circle-up`, `chevron-double-down`, `chevron-double-left`, `chevron-double-right`, `chevron-double-up`, `chevron-down`, `chevron-left`, `chevron-right`, `chevron-square-down`, `chevron-square-left`, `chevron-square-right`, `chevron-square-up`, `chevron-up`, `child`, `chimney`, `chrome`, `chromecast`, `church`, `circle`, `circle-notch`, `city`, `clarinet`, `claw-marks`, `clinic-medical`, `clipboard`, `clipboard-check`, `clipboard-list`, `clipboard-list-check`, `clipboard-prescription`, `clipboard-user`, `clock`, `clone`, `closed-captioning`, `cloud`, `cloud-download`, `cloud-download-alt`, `cloud-drizzle`, `cloud-hail`, `cloud-hail-mixed`, `cloud-meatball`, `cloud-moon`, `cloud-moon-rain`, `cloud-music`, `cloud-rain`, `cloud-rainbow`, `cloud-showers`, `cloud-showers-heavy`, `cloud-sleet`, `cloud-snow`, `cloud-sun`, `cloud-sun-rain`, `cloud-upload`, `cloud-upload-alt`, `clouds`, `clouds-moon`, `clouds-sun`, `cloudscale`, `cloudsmith`, `cloudversify`, `club`, `clubs`, `cocktail`, `code`, `code-branch`, `code-commit`, `code-merge`, `codepen`, `codiepie`, `coffee`, `coffee-pot`, `coffee-togo`, `coffin`, `coffin-cross`, `cog`, `cogs`, `coin`, `coins`, `columns`, `comet`, `comment`, `comment-alt`, `comment-alt-check`, `comment-alt-dollar`, `comment-alt-dots`, `comment-alt-edit`, `comment-alt-exclamation`, `comment-alt-lines`, `comment-alt-medical`, `comment-alt-minus`, `comment-alt-music`, `comment-alt-plus`, `comment-alt-slash`, `comment-alt-smile`, `comment-alt-times`, `comment-check`, `comment-dollar`, `comment-dots`, `comment-edit`, `comment-exclamation`, `comment-lines`, `comment-medical`, `comment-minus`, `comment-music`, `comment-plus`, `comment-slash`, `comment-smile`, `comment-times`, `comments`, `comments-alt`, `comments-alt-dollar`, `comments-dollar`, `compact-disc`, `compass`, `compass-slash`, `compress`, `compress-alt`, `compress-arrows-alt`, `compress-wide`, `computer-classic`, `computer-speaker`, `concierge-bell`, `confluence`, `connectdevelop`, `construction`, `container-storage`, `contao`, `conveyor-belt`, `conveyor-belt-alt`, `cookie`, `cookie-bite`, `copy`, `copyright`, `corn`, `couch`, `cow`, `cowbell`, `cowbell-more`, `cpanel`, `creative-commons`, `creative-commons-by`, `creative-commons-nc`, `creative-commons-nc-eu`, `creative-commons-nc-jp`, `creative-commons-nd`, `creative-commons-pd`, `creative-commons-pd-alt`, `creative-commons-remix`, `creative-commons-sa`, `creative-commons-sampling`, `creative-commons-sampling-plus`, `creative-commons-share`, `creative-commons-zero`, `credit-card`, `credit-card-blank`, `credit-card-front`, `cricket`, `critical-role`, `croissant`, `crop`, `crop-alt`, `cross`, `crosshairs`, `crow`, `crown`, `crutch`, `crutches`, `css3`, `css3-alt`, `cube`, `cubes`, `curling`, `cut`, `cuttlefish`, `d-and-d`, `d-and-d-beyond`, `dagger`, `dailymotion`, `dashcube`, `database`, `deaf`, `debug`, `deer`, `deer-rudolph`, `deezer`, `delicious`, `democrat`, `deploydog`, `deskpro`, `desktop`, `desktop-alt`, `dev`, `deviantart`, `dewpoint`, `dharmachakra`, `dhl`, `diagnoses`, `diamond`, `diaspora`, `dice`, `dice-d10`, `dice-d12`, `dice-d20`, `dice-d4`, `dice-d6`, `dice-d8`, `dice-five`, `dice-four`, `dice-one`, `dice-six`, `dice-three`, `dice-two`, `digg`, `digital-ocean`, `digital-tachograph`, `diploma`, `directions`, `disc-drive`, `discord`, `discourse`, `disease`, `dish`, `divide`, `dizzy`, `dna`, `do-not-enter`, `dochub`, `docker`, `dog`, `dog-leashed`, `dollar-sign`, `dolly`, `dolly-empty`, `dolly-flatbed`, `dolly-flatbed-alt`, `dolly-flatbed-empty`, `donate`, `door-closed`, `door-open`, `dot-circle`, `dove`, `download`, `draft2digital`, `drafting-compass`, `dragon`, `draw-circle`, `draw-polygon`, `draw-square`, `dreidel`, `dribbble`, `dribbble-square`, `drone`, `drone-alt`, `dropbox`, `drum`, `drum-steelpan`, `drumstick`, `drumstick-bite`, `drupal`, `dryer`, `dryer-alt`, `duck`, `dumbbell`, `dumpster`, `dumpster-fire`, `dungeon`, `dyalog`, `ear`, `ear-muffs`, `earlybirds`, `ebay`, `eclipse`, `eclipse-alt`, `edge`, `edge-legacy`, `edit`, `egg`, `egg-fried`, `eject`, `elementor`, `elephant`, `ellipsis-h`, `ellipsis-h-alt`, `ellipsis-v`, `ellipsis-v-alt`, `ello`, `ember`, `empire`, `empty-set`, `engine-warning`, `envelope`, `envelope-open`, `envelope-open-dollar`, `envelope-open-text`, `envelope-square`, `envira`, `equals`, `eraser`, `erlang`, `ethereum`, `ethernet`, `etsy`, `euro-sign`, `evernote`, `exchange`, `exchange-alt`, `exclamation`, `exclamation-circle`, `exclamation-square`, `exclamation-triangle`, `expand`, `expand-alt`, `expand-arrows`, `expand-arrows-alt`, `expand-wide`, `expeditedssl`, `external-link`, `external-link-alt`, `external-link-square`, `external-link-square-alt`, `eye`, `eye-dropper`, `eye-evil`, `eye-slash`, `facebook`, `facebook-f`, `facebook-messenger`, `facebook-square`, `fan`, `fan-table`, `fantasy-flight-games`, `farm`, `fast-backward`, `fast-forward`, `faucet`, `faucet-drip`, `fax`, `feather`, `feather-alt`, `fedex`, `fedora`, `female`, `field-hockey`, `figma`, `file`, `file-alt`, `file-archive`, `file-audio`, `file-certificate`, `file-chart-line`, `file-chart-pie`, `file-check`, `file-code`, `file-contract`, `file-csv`, `file-download`, `file-edit`, `file-excel`, `file-exclamation`, `file-export`, `file-image`, `file-import`, `file-invoice`, `file-invoice-dollar`, `file-medical`, `file-medical-alt`, `file-minus`, `file-music`, `file-pdf`, `file-plus`, `file-powerpoint`, `file-prescription`, `file-search`, `file-signature`, `file-spreadsheet`, `file-times`, `file-upload`, `file-user`, `file-video`, `file-word`, `files-medical`, `fill`, `fill-drip`, `film`, `film-alt`, `film-canister`, `filter`, `fingerprint`, `fire`, `fire-alt`, `fire-extinguisher`, `fire-smoke`, `firefox`, `firefox-browser`, `fireplace`, `first-aid`, `first-order`, `first-order-alt`, `firstdraft`, `fish`, `fish-cooked`, `fist-raised`, `flag`, `flag-alt`, `flag-checkered`, `flag-usa`, `flame`, `flashlight`, `flask`, `flask-poison`, `flask-potion`, `flickr`, `flipboard`, `flower`, `flower-daffodil`, `flower-tulip`, `flushed`, `flute`, `flux-capacitor`, `fly`, `fog`, `folder`, `folder-download`, `folder-minus`, `folder-open`, `folder-plus`, `folder-times`, `folder-tree`, `folder-upload`, `folders`, `font`, `font-awesome`, `font-awesome-alt`, `font-awesome-flag`, `font-awesome-logo-full`, `font-case`, `fonticons`, `fonticons-fi`, `football-ball`, `football-helmet`, `forklift`, `fort-awesome`, `fort-awesome-alt`, `forumbee`, `forward`, `foursquare`, `fragile`, `free-code-camp`, `freebsd`, `french-fries`, `frog`, `frosty-head`, `frown`, `frown-open`, `fulcrum`, `function`, `funnel-dollar`, `furniture-couch`, `futbol`, `galactic-republic`, `galactic-senate`, `galaxy`, `game-board`, `game-board-alt`, `game-console-handheld`, `gamepad`, `gamepad-alt`, `garage`, `garage-car`, `garage-open`, `gas-pump`, `gas-pump-slash`, `gavel`, `gem`, `genderless`, `get-pocket`, `gg`, `gg-circle`, `ghost`, `gift`, `gift-card`, `gifts`, `gingerbread-man`, `git`, `git-alt`, `git-square`, `github`, `github-alt`, `github-square`, `gitlab`, `gitter`, `glass`, `glass-champagne`, `glass-cheers`, `glass-citrus`, `glass-martini`, `glass-martini-alt`, `glass-whiskey`, `glass-whiskey-rocks`, `glasses`, `glasses-alt`, `glide`, `glide-g`, `globe`, `globe-africa`, `globe-americas`, `globe-asia`, `globe-europe`, `globe-snow`, `globe-stand`, `gofore`, `golf-ball`, `golf-club`, `goodreads`, `goodreads-g`, `google`, `google-drive`, `google-pay`, `google-play`, `google-plus`, `google-plus-g`, `google-plus-square`, `google-wallet`, `gopuram`, `graduation-cap`, `gramophone`, `gratipay`, `grav`, `greater-than`, `greater-than-equal`, `grimace`, `grin`, `grin-alt`, `grin-beam`, `grin-beam-sweat`, `grin-hearts`, `grin-squint`, `grin-squint-tears`, `grin-stars`, `grin-tears`, `grin-tongue`, `grin-tongue-squint`, `grin-tongue-wink`, `grin-wink`, `grip-horizontal`, `grip-lines`, `grip-lines-vertical`, `grip-vertical`, `gripfire`, `grunt`, `guilded`, `guitar`, `guitar-electric`, `guitars`, `gulp`, `h-square`, `h1`, `h2`, `h3`, `h4`, `h5`, `h6`, `hacker-news`, `hacker-news-square`, `hackerrank`, `hamburger`, `hammer`, `hammer-war`, `hamsa`, `hand-heart`, `hand-holding`, `hand-holding-box`, `hand-holding-heart`, `hand-holding-magic`, `hand-holding-medical`, `hand-holding-seedling`, `hand-holding-usd`, `hand-holding-water`, `hand-lizard`, `hand-middle-finger`, `hand-paper`, `hand-peace`, `hand-point-down`, `hand-point-left`, `hand-point-right`, `hand-point-up`, `hand-pointer`, `hand-receiving`, `hand-rock`, `hand-scissors`, `hand-sparkles`, `hand-spock`, `hands`, `hands-heart`, `hands-helping`, `hands-usd`, `hands-wash`, `handshake`, `handshake-alt`, `handshake-alt-slash`, `handshake-slash`, `hanukiah`, `hard-hat`, `hashtag`, `hat-chef`, `hat-cowboy`, `hat-cowboy-side`, `hat-santa`, `hat-winter`, `hat-witch`, `hat-wizard`, `hdd`, `head-side`, `head-side-brain`, `head-side-cough`, `head-side-cough-slash`, `head-side-headphones`, `head-side-mask`, `head-side-medical`, `head-side-virus`, `head-vr`, `heading`, `headphones`, `headphones-alt`, `headset`, `heart`, `heart-broken`, `heart-circle`, `heart-rate`, `heart-square`, `heartbeat`, `heat`, `helicopter`, `helmet-battle`, `hexagon`, `highlighter`, `hiking`, `hippo`, `hips`, `hire-a-helper`, `history`, `hive`, `hockey-mask`, `hockey-puck`, `hockey-sticks`, `holly-berry`, `home`, `home-alt`, `home-heart`, `home-lg`, `home-lg-alt`, `hood-cloak`, `hooli`, `horizontal-rule`, `hornbill`, `horse`, `horse-head`, `horse-saddle`, `hospital`, `hospital-alt`, `hospital-symbol`, `hospital-user`, `hospitals`, `hot-tub`, `hotdog`, `hotel`, `hotjar`, `hourglass`, `hourglass-end`, `hourglass-half`, `hourglass-start`, `house`, `house-damage`, `house-day`, `house-flood`, `house-leave`, `house-night`, `house-return`, `house-signal`, `house-user`, `houzz`, `hryvnia`, `html5`, `hubspot`, `humidity`, `hurricane`, `i-cursor`, `ice-cream`, `ice-skate`, `icicles`, `icons`, `icons-alt`, `id-badge`, `id-card`, `id-card-alt`, `ideal`, `igloo`, `image`, `image-polaroid`, `images`, `imdb`, `inbox`, `inbox-in`, `inbox-out`, `indent`, `industry`, `industry-alt`, `infinity`, `info`, `info-circle`, `info-square`, `inhaler`, `instagram`, `instagram-square`, `instalod`, `integral`, `intercom`, `internet-explorer`, `intersection`, `inventory`, `invision`, `ioxhost`, `island-tropical`, `italic`, `itch-io`, `itunes`, `itunes-note`, `jack-o-lantern`, `java`, `jedi`, `jedi-order`, `jenkins`, `jira`, `joget`, `joint`, `joomla`, `journal-whills`, `joystick`, `js`, `js-square`, `jsfiddle`, `jug`, `kaaba`, `kaggle`, `kazoo`, `keg`, `kennel`, `key`, `key-skeleton`, `keybase`, `keyboard`, `keycdn`, `keynote`, `khanda`, `kickstarter`, `kickstarter-k`, `kidneys`, `kiss`, `kiss-beam`, `kiss-wink-heart`, `kite`, `kiwi-bird`, `kiwi-fruit`, `knife-kitchen`, `korvue`, `lambda`, `lamp`, `lamp-desk`, `lamp-floor`, `landmark`, `landmark-alt`, `language`, `laptop`, `laptop-code`, `laptop-house`, `laptop-medical`, `laravel`, `lasso`, `lastfm`, `lastfm-square`, `laugh`, `laugh-beam`, `laugh-squint`, `laugh-wink`, `layer-group`, `layer-minus`, `layer-plus`, `leaf`, `leaf-heart`, `leaf-maple`, `leaf-oak`, `leanpub`, `lemon`, `less`, `less-than`, `less-than-equal`, `level-down`, `level-down-alt`, `level-up`, `level-up-alt`, `life-ring`, `light-ceiling`, `light-switch`, `light-switch-off`, `light-switch-on`, `lightbulb`, `lightbulb-dollar`, `lightbulb-exclamation`, `lightbulb-on`, `lightbulb-slash`, `lights-holiday`, `line`, `link`, `linkedin`, `linkedin-in`, `linode`, `linux`, `lips`, `lira-sign`, `list`, `list-alt`, `list-music`, `list-ol`, `list-ul`, `location`, `location-arrow`, `location-circle`, `location-slash`, `lock`, `lock-alt`, `lock-open`, `lock-open-alt`, `long-arrow-alt-down`, `long-arrow-alt-left`, `long-arrow-alt-right`, `long-arrow-alt-up`, `long-arrow-down`, `long-arrow-left`, `long-arrow-right`, `long-arrow-up`, `loveseat`, `low-vision`, `luchador`, `luggage-cart`, `lungs`, `lungs-virus`, `lyft`, `mace`, `magento`, `magic`, `magnet`, `mail-bulk`, `mailbox`, `mailchimp`, `male`, `mandalorian`, `mandolin`, `map`, `map-marked`, `map-marked-alt`, `map-marker`, `map-marker-alt`, `map-marker-alt-slash`, `map-marker-check`, `map-marker-edit`, `map-marker-exclamation`, `map-marker-minus`, `map-marker-plus`, `map-marker-question`, `map-marker-slash`, `map-marker-smile`, `map-marker-times`, `map-pin`, `map-signs`, `markdown`, `marker`, `mars`, `mars-double`, `mars-stroke`, `mars-stroke-h`, `mars-stroke-v`, `mask`, `mastodon`, `maxcdn`, `mdb`, `meat`, `medal`, `medapps`, `medium`, `medium-m`, `medkit`, `medrt`, `meetup`, `megaphone`, `megaport`, `meh`, `meh-blank`, `meh-rolling-eyes`, `memory`, `mendeley`, `menorah`, `mercury`, `merge`, `messenger`, `meteor`, `microblog`, `microchip`, `microphone`, `microphone-alt`, `microphone-alt-slash`, `microphone-slash`, `microphone-stand`, `microscope`, `microsoft`, `microwave`, `mind-share`, `minus`, `minus-circle`, `minus-hexagon`, `minus-octagon`, `minus-square`, `mistletoe`, `mitten`, `mix`, `mixcloud`, `mixer`, `mizuni`, `mobile`, `mobile-alt`, `mobile-android`, `mobile-android-alt`, `modx`, `monero`, `money-bill`, `money-bill-alt`, `money-bill-wave`, `money-bill-wave-alt`, `money-check`, `money-check-alt`, `money-check-edit`, `money-check-edit-alt`, `monitor-heart-rate`, `monkey`, `monument`, `moon`, `moon-cloud`, `moon-stars`, `mortar-pestle`, `mosque`, `motorcycle`, `mountain`, `mountains`, `mouse`, `mouse-alt`, `mouse-pointer`, `mp3-player`, `mug`, `mug-hot`, `mug-marshmallows`, `mug-tea`, `music`, `music-alt`, `music-alt-slash`, `music-slash`, `napster`, `narwhal`, `neos`, `network-wired`, `neuter`, `newspaper`, `nimblr`, `nintendo-switch`, `node`, `node-js`, `not-equal`, `notes-medical`, `npm`, `ns8`, `nutritionix`, `object-group`, `object-ungroup`, `octagon`, `odnoklassniki`, `odnoklassniki-square`, `oil-can`, `oil-temp`, `old-republic`, `om`, `omega`, `opencart`, `openid`, `opera`, `optin-monster`, `orcid`, `ornament`, `osi`, `otter`, `outdent`, `outlet`, `oven`, `overline`, `page-break`, `page4`, `pagelines`, `pager`, `paint-brush`, `paint-brush-alt`, `paint-roller`, `palette`, `palfed`, `pallet`, `pallet-alt`, `paper-plane`, `paperclip`, `parachute-box`, `paragraph`, `paragraph-rtl`, `parking`, `parking-circle`, `parking-circle-slash`, `parking-slash`, `passport`, `pastafarianism`, `paste`, `patreon`, `pause`, `pause-circle`, `paw`, `paw-alt`, `paw-claws`, `paypal`, `peace`, `pegasus`, `pen`, `pen-alt`, `pen-fancy`, `pen-nib`, `pen-square`, `pencil`, `pencil-alt`, `pencil-paintbrush`, `pencil-ruler`, `pennant`, `penny-arcade`, `people-arrows`, `people-carry`, `pepper-hot`, `perbyte`, `percent`, `percentage`, `periscope`, `person-booth`, `person-carry`, `person-dolly`, `person-dolly-empty`, `person-sign`, `phabricator`, `phoenix-framework`, `phoenix-squadron`, `phone`, `phone-alt`, `phone-laptop`, `phone-office`, `phone-plus`, `phone-rotary`, `phone-slash`, `phone-square`, `phone-square-alt`, `phone-volume`, `photo-video`, `php`, `pi`, `piano`, `piano-keyboard`, `pie`, `pied-piper`, `pied-piper-alt`, `pied-piper-hat`, `pied-piper-pp`, `pied-piper-square`, `pig`, `piggy-bank`, `pills`, `pinterest`, `pinterest-p`, `pinterest-square`, `pizza`, `pizza-slice`, `place-of-worship`, `plane`, `plane-alt`, `plane-arrival`, `plane-departure`, `plane-slash`, `planet-moon`, `planet-ringed`, `play`, `play-circle`, `playstation`, `plug`, `plus`, `plus-circle`, `plus-hexagon`, `plus-octagon`, `plus-square`, `podcast`, `podium`, `podium-star`, `police-box`, `poll`, `poll-h`, `poll-people`, `poo`, `poo-storm`, `poop`, `popcorn`, `portal-enter`, `portal-exit`, `portrait`, `pound-sign`, `power-off`, `pray`, `praying-hands`, `prescription`, `prescription-bottle`, `prescription-bottle-alt`, `presentation`, `print`, `print-search`, `print-slash`, `procedures`, `product-hunt`, `project-diagram`, `projector`, `pump-medical`, `pump-soap`, `pumpkin`, `pushed`, `puzzle-piece`, `python`, `qq`, `qrcode`, `question`, `question-circle`, `question-square`, `quidditch`, `quinscape`, `quora`, `quote-left`, `quote-right`, `quran`, `r-project`, `rabbit`, `rabbit-fast`, `racquet`, `radar`, `radiation`, `radiation-alt`, `radio`, `radio-alt`, `rainbow`, `raindrops`, `ram`, `ramp-loading`, `random`, `raspberry-pi`, `ravelry`, `raygun`, `react`, `reacteurope`, `readme`, `rebel`, `receipt`, `record-vinyl`, `rectangle-landscape`, `rectangle-portrait`, `rectangle-wide`, `recycle`, `red-river`, `reddit`, `reddit-alien`, `reddit-square`, `redhat`, `redo`, `redo-alt`, `refrigerator`, `registered`, `remove-format`, `renren`, `repeat`, `repeat-1`, `repeat-1-alt`, `repeat-alt`, `replace`, `replyd`, `republican`, `researchgate`, `resolving`, `restroom`, `retweet`, `retweet-alt`, `rev`, `ribbon`, `ring`, `rings-wedding`, `road`, `robot`, `rocket`, `rocketchat`, `rockrms`, `route`, `route-highway`, `route-interstate`, `router`, `rss`, `rss-square`, `ruble-sign`, `ruler`, `ruler-combined`, `ruler-horizontal`, `ruler-triangle`, `ruler-vertical`, `running`, `rupee-sign`, `rust`, `rv`, `sack`, `sack-dollar`, `sad-cry`, `sad-tear`, `safari`, `salad`, `salesforce`, `sandwich`, `sass`, `satellite`, `satellite-dish`, `sausage`, `save`, `sax-hot`, `saxophone`, `scalpel`, `scalpel-path`, `scanner`, `scanner-image`, `scanner-keyboard`, `scanner-touchscreen`, `scarecrow`, `scarf`, `schlix`, `school`, `screwdriver`, `scribd`, `scroll`, `scroll-old`, `scrubber`, `scythe`, `sd-card`, `search`, `search-dollar`, `search-location`, `search-minus`, `search-plus`, `searchengin`, `seedling`, `sellcast`, `sellsy`, `send-back`, `send-backward`, `sensor`, `sensor-alert`, `sensor-fire`, `sensor-on`, `sensor-smoke`, `server`, `servicestack`, `shapes`, `share`, `share-all`, `share-alt`, `share-alt-square`, `share-square`, `sheep`, `shekel-sign`, `shield`, `shield-alt`, `shield-check`, `shield-cross`, `shield-virus`, `ship`, `shipping-fast`, `shipping-timed`, `shirtsinbulk`, `shish-kebab`, `shoe-prints`, `shopify`, `shopping-bag`, `shopping-basket`, `shopping-cart`, `shopware`, `shovel`, `shovel-snow`, `shower`, `shredder`, `shuttle-van`, `shuttlecock`, `sickle`, `sigma`, `sign`, `sign-in`, `sign-in-alt`, `sign-language`, `sign-out`, `sign-out-alt`, `signal`, `signal-1`, `signal-2`, `signal-3`, `signal-4`, `signal-alt`, `signal-alt-1`, `signal-alt-2`, `signal-alt-3`, `signal-alt-slash`, `signal-slash`, `signal-stream`, `signature`, `sim-card`, `simplybuilt`, `sink`, `siren`, `siren-on`, `sistrix`, `sitemap`, `sith`, `skating`, `skeleton`, `sketch`, `ski-jump`, `ski-lift`, `skiing`, `skiing-nordic`, `skull`, `skull-cow`, `skull-crossbones`, `skyatlas`, `skype`, `slack`, `slack-hash`, `slash`, `sledding`, `sleigh`, `sliders-h`, `sliders-h-square`, `sliders-v`, `sliders-v-square`, `slideshare`, `smile`, `smile-beam`, `smile-plus`, `smile-wink`, `smog`, `smoke`, `smoking`, `smoking-ban`, `sms`, `snake`, `snapchat`, `snapchat-ghost`, `snapchat-square`, `snooze`, `snow-blowing`, `snowboarding`, `snowflake`, `snowflakes`, `snowman`, `snowmobile`, `snowplow`, `soap`, `socks`, `solar-panel`, `solar-system`, `sort`, `sort-alpha-down`, `sort-alpha-down-alt`, `sort-alpha-up`, `sort-alpha-up-alt`, `sort-alt`, `sort-amount-down`, `sort-amount-down-alt`, `sort-amount-up`, `sort-amount-up-alt`, `sort-circle`, `sort-circle-down`, `sort-circle-up`, `sort-down`, `sort-numeric-down`, `sort-numeric-down-alt`, `sort-numeric-up`, `sort-numeric-up-alt`, `sort-shapes-down`, `sort-shapes-down-alt`, `sort-shapes-up`, `sort-shapes-up-alt`, `sort-size-down`, `sort-size-down-alt`, `sort-size-up`, `sort-size-up-alt`, `sort-up`, `soundcloud`, `soup`, `sourcetree`, `spa`, `space-shuttle`, `space-station-moon`, `space-station-moon-alt`, `spade`, `sparkles`, `speakap`, `speaker-deck`, `speakers`, `spell-check`, `spider`, `spider-black-widow`, `spider-web`, `spinner`, `spinner-third`, `splotch`, `spotify`, `spray-can`, `sprinkler`, `square`, `square-full`, `square-root`, `square-root-alt`, `squarespace`, `squirrel`, `stack-exchange`, `stack-overflow`, `stackpath`, `staff`, `stamp`, `star`, `star-and-crescent`, `star-christmas`, `star-exclamation`, `star-half`, `star-half-alt`, `star-of-david`, `star-of-life`, `star-shooting`, `starfighter`, `starfighter-alt`, `stars`, `starship`, `starship-freighter`, `staylinked`, `steak`, `steam`, `steam-square`, `steam-symbol`, `steering-wheel`, `step-backward`, `step-forward`, `stethoscope`, `sticker-mule`, `sticky-note`, `stocking`, `stomach`, `stop`, `stop-circle`, `stopwatch`, `stopwatch-20`, `store`, `store-alt`, `store-alt-slash`, `store-slash`, `strava`, `stream`, `street-view`, `stretcher`, `strikethrough`, `stripe`, `stripe-s`, `stroopwafel`, `studiovinari`, `stumbleupon`, `stumbleupon-circle`, `subscript`, `subway`, `suitcase`, `suitcase-rolling`, `sun`, `sun-cloud`, `sun-dust`, `sun-haze`, `sunglasses`, `sunrise`, `sunset`, `superpowers`, `superscript`, `supple`, `surprise`, `suse`, `swatchbook`, `swift`, `swimmer`, `swimming-pool`, `sword`, `sword-laser`, `sword-laser-alt`, `swords`, `swords-laser`, `symfony`, `synagogue`, `sync`, `sync-alt`, `syringe`, `table`, `table-tennis`, `tablet`, `tablet-alt`, `tablet-android`, `tablet-android-alt`, `tablet-rugged`, `tablets`, `tachometer`, `tachometer-alt`, `tachometer-alt-average`, `tachometer-alt-fast`, `tachometer-alt-fastest`, `tachometer-alt-slow`, `tachometer-alt-slowest`, `tachometer-average`, `tachometer-fast`, `tachometer-fastest`, `tachometer-slow`, `tachometer-slowest`, `taco`, `tag`, `tags`, `tally`, `tanakh`, `tape`, `tasks`, `tasks-alt`, `taxi`, `teamspeak`, `teeth`, `teeth-open`, `telegram`, `telegram-plane`, `telescope`, `temperature-down`, `temperature-frigid`, `temperature-high`, `temperature-hot`, `temperature-low`, `temperature-up`, `tencent-weibo`, `tenge`, `tennis-ball`, `terminal`, `text`, `text-height`, `text-size`, `text-width`, `th`, `th-large`, `th-list`, `the-red-yeti`, `theater-masks`, `themeco`, `themeisle`, `thermometer`, `thermometer-empty`, `thermometer-full`, `thermometer-half`, `thermometer-quarter`, `thermometer-three-quarters`, `theta`, `think-peaks`, `thumbs-down`, `thumbs-up`, `thumbtack`, `thunderstorm`, `thunderstorm-moon`, `thunderstorm-sun`, `ticket`, `ticket-alt`, `tiktok`, `tilde`, `times`, `times-circle`, `times-hexagon`, `times-octagon`, `times-square`, `tint`, `tint-slash`, `tire`, `tire-flat`, `tire-pressure-warning`, `tire-rugged`, `tired`, `toggle-off`, `toggle-on`, `toilet`, `toilet-paper`, `toilet-paper-alt`, `toilet-paper-slash`, `tombstone`, `tombstone-alt`, `toolbox`, `tools`, `tooth`, `toothbrush`, `torah`, `torii-gate`, `tornado`, `tractor`, `trade-federation`, `trademark`, `traffic-cone`, `traffic-light`, `traffic-light-go`, `traffic-light-slow`, `traffic-light-stop`, `trailer`, `train`, `tram`, `transgender`, `transgender-alt`, `transporter`, `transporter-1`, `transporter-2`, `transporter-3`, `transporter-empty`, `trash`, `trash-alt`, `trash-restore`, `trash-restore-alt`, `trash-undo`, `trash-undo-alt`, `treasure-chest`, `tree`, `tree-alt`, `tree-christmas`, `tree-decorated`, `tree-large`, `tree-palm`, `trees`, `trello`, `triangle`, `triangle-music`, `tripadvisor`, `trophy`, `trophy-alt`, `truck`, `truck-container`, `truck-couch`, `truck-loading`, `truck-monster`, `truck-moving`, `truck-pickup`, `truck-plow`, `truck-ramp`, `trumpet`, `tshirt`, `tty`, `tumblr`, `tumblr-square`, `turkey`, `turntable`, `turtle`, `tv`, `tv-alt`, `tv-music`, `tv-retro`, `twitch`, `twitter`, `twitter-square`, `typewriter`, `typo3`, `uber`, `ubuntu`, `ufo`, `ufo-beam`, `uikit`, `umbraco`, `umbrella`, `umbrella-beach`, `underline`, `undo`, `undo-alt`, `unicorn`, `union`, `uniregistry`, `unity`, `universal-access`, `university`, `unlink`, `unlock`, `unlock-alt`, `unsplash`, `untappd`, `upload`, `ups`, `usb`, `usb-drive`, `usd-circle`, `usd-square`, `user`, `user-alien`, `user-alt`, `user-alt-slash`, `user-astronaut`, `user-chart`, `user-check`, `user-circle`, `user-clock`, `user-cog`, `user-cowboy`, `user-crown`, `user-edit`, `user-friends`, `user-graduate`, `user-hard-hat`, `user-headset`, `user-injured`, `user-lock`, `user-md`, `user-md-chat`, `user-minus`, `user-music`, `user-ninja`, `user-nurse`, `user-plus`, `user-robot`, `user-secret`, `user-shield`, `user-slash`, `user-tag`, `user-tie`, `user-times`, `user-unlock`, `user-visor`, `users`, `users-class`, `users-cog`, `users-crown`, `users-medical`, `users-slash`, `usps`, `ussunnah`, `utensil-fork`, `utensil-knife`, `utensil-spoon`, `utensils`, `utensils-alt`, `vaadin`, `vacuum`, `vacuum-robot`, `value-absolute`, `vector-square`, `venus`, `venus-double`, `venus-mars`, `vhs`, `viacoin`, `viadeo`, `viadeo-square`, `vial`, `vials`, `viber`, `video`, `video-plus`, `video-slash`, `vihara`, `vimeo`, `vimeo-square`, `vimeo-v`, `vine`, `violin`, `virus`, `virus-slash`, `viruses`, `vk`, `vnv`, `voicemail`, `volcano`, `volleyball-ball`, `volume`, `volume-down`, `volume-mute`, `volume-off`, `volume-slash`, `volume-up`, `vote-nay`, `vote-yea`, `vr-cardboard`, `vuejs`, `wagon-covered`, `walker`, `walkie-talkie`, `walking`, `wallet`, `wand`, `wand-magic`, `warehouse`, `warehouse-alt`, `washer`, `watch`, `watch-calculator`, `watch-fitness`, `water`, `water-lower`, `water-rise`, `wave-sine`, `wave-square`, `wave-triangle`, `waveform`, `waveform-path`, `waze`, `webcam`, `webcam-slash`, `weebly`, `weibo`, `weight`, `weight-hanging`, `weixin`, `whale`, `whatsapp`, `whatsapp-square`, `wheat`, `wheelchair`, `whistle`, `whmcs`, `wifi`, `wifi-1`, `wifi-2`, `wifi-slash`, `wikipedia-w`, `wind`, `wind-turbine`, `wind-warning`, `window`, `window-alt`, `window-close`, `window-frame`, `window-frame-open`, `window-maximize`, `window-minimize`, `window-restore`, `windows`, `windsock`, `wine-bottle`, `wine-glass`, `wine-glass-alt`, `wix`, `wizards-of-the-coast`, `wodu`, `wolf-pack-battalion`, `won-sign`, `wordpress`, `wordpress-simple`, `wpbeginner`, `wpexplorer`, `wpforms`, `wpressr`, `wreath`, `wrench`, `x-ray`, `xbox`, `xing`, `xing-square`, `y-combinator`, `yahoo`, `yammer`, `yandex`, `yandex-international`, `yarn`, `yelp`, `yen-sign`, `yin-yang`, `yoast`, `youtube`, `youtube-square`, `zhihu`, `zap`

</details>

## Icon Selection Best Practices

1. **Consistency**: Use icons from the same style family throughout your plugin
2. **Clarity**: Choose icons that clearly represent their function
3. **Context**: Consider the DatoCMS UI conventions when selecting icons
4. **Fallbacks**: Always have a text label alongside icons for accessibility

## Common Icon Patterns

### Action Icons
```typescript
// CRUD operations
const createIcon: Icon = 'plus';
const readIcon: Icon = 'eye';
const updateIcon: Icon = 'edit';
const deleteIcon: Icon = 'trash';

// Navigation
const backIcon: Icon = 'arrow-left';
const forwardIcon: Icon = 'arrow-right';
const menuIcon: Icon = 'bars';
const closeIcon: Icon = 'times';
```

### Status Icons
```typescript
// States
const successIcon: Icon = 'check-circle';
const errorIcon: Icon = 'times-circle';
const warningIcon: Icon = 'exclamation-triangle';
const infoIcon: Icon = 'info-circle';

// Loading/Progress
const loadingIcon: Icon = 'spinner';
const syncIcon: Icon = 'sync';
const refreshIcon: Icon = 'redo';
```

### Content Type Icons
```typescript
// Media
const imageIcon: Icon = 'image';
const videoIcon: Icon = 'video';
const audioIcon: Icon = 'music';
const documentIcon: Icon = 'file-alt';

// Data
const databaseIcon: Icon = 'database';
const tableIcon: Icon = 'table';
const chartIcon: Icon = 'chart-bar';
```

## Custom SVG Icons

When Font Awesome doesn't have the icon you need, use custom SVG:

```typescript
const customBrandIcon: Icon = {
  type: 'svg',
  viewBox: '0 0 24 24',
  content: `
    <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2z"/>
    <circle cx="12" cy="12" r="3" fill="white"/>
  `
};

// Using with proper formatting
const formattedIcon: Icon = {
  type: 'svg',
  viewBox: '0 0 100 100',
  content: '<rect x="10" y="10" width="80" height="80" rx="10"/>'
};
```

### SVG Best Practices

1. **ViewBox**: Always specify a viewBox for proper scaling
2. **Simplicity**: Keep SVG paths simple for better performance
3. **Colors**: Avoid hardcoded colors; let the theme system handle them
4. **Size**: Keep SVG content concise (under 1KB ideally)

## TypeScript Usage

```typescript
import type { Icon } from 'datocms-plugin-sdk';

// Type-safe icon selection
function getIconForFieldType(fieldType: string): Icon {
  const iconMap: Record<string, Icon> = {
    string: 'font',
    text: 'align-left',
    boolean: 'toggle-on',
    integer: 'hashtag',
    float: 'percentage',
    date: 'calendar',
    datetime: 'clock',
    color: 'palette',
    json: 'code',
    seo: 'search',
    file: 'paperclip',
    gallery: 'images'
  };
  
  return iconMap[fieldType] || 'question-circle';
}

// Icon with fallback
function renderIcon(icon: Icon | undefined): Icon {
  return icon || 'cube'; // Default fallback
}
```

## Integration Examples

### In Dropdown Actions
```typescript
connect({
  itemFormDropdownActions() {
    return [
      {
        id: 'export',
        label: 'Export',
        icon: 'download',
        actions: [
          { id: 'csv', label: 'Export as CSV', icon: 'file-csv' },
          { id: 'json', label: 'Export as JSON', icon: 'file-code' }
        ]
      }
    ];
  }
});
```

### In Navigation
```typescript
connect({
  mainNavigationTabs() {
    return [
      {
        id: 'analytics',
        label: 'Analytics',
        icon: 'chart-line',
        pointsTo: {
          pageId: 'analytics-dashboard'
        }
      }
    ];
  }
});
```

### In Sidebar Items
```typescript
connect({
  contentAreaSidebarItems() {
    return [
      {
        id: 'tools',
        label: 'Tools',
        icon: 'toolbox',
        pointsTo: {
          pageId: 'tools-page'
        }
      }
    ];
  }
});
```

## Complete Icon List

For a comprehensive list of all 4,205 available Font Awesome icon identifiers, see [Font Awesome Icons](./font-awesome-icons.md).

## Related Documentation

- [Font Awesome Icons](./font-awesome-icons.md) - Complete list of all icon identifiers
- [Shared Types Reference](./shared-types.md)
- [UI Components](../03-ui-components/README.md)
- [Hook Documentation](../02-hooks/README.md)