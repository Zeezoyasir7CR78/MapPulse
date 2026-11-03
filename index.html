/**
 * MapPulse — a split-screen mapping application.
 *
 * Architecture overview
 * ---------------------
 *  types           → shared domain types (Place, Category, Route, Step)
 *  data            → mock POI dataset
 *  geo             → pure geospatial helpers (distance, bearing, interpolation)
 *  machine         → useReducer-based app state machine
 *  hooks           → useBookmarks (localStorage), useRouteCalculator, useDebounced
 *  components      → Sidebar, SearchBar, CategoryOverlay, PlaceCard, RoutePanel,
 *                    FavoritesList, MapView (react-leaflet), SvgMarker
 */

import {
  useCallback,
  useEffect,
  useMemo,
  useReducer,
  useRef,
  useState,
  type FC,
  type KeyboardEvent,
  type ReactNode,
} from "react";
import {
  MapContainer,
  TileLayer,
  Marker,
  Polyline,
  Popup,
  useMap,
  useMapEvents,
} from "react-leaflet";
import L, { type LatLngExpression, type LatLngTuple } from "leaflet";
import {
  Bookmark,
  BookmarkCheck,
  Coffee,
  Utensils,
  Trees,
  MapPin,
  Search,
  X,
  Navigation,
  ArrowLeft,
  Clock,
  Ruler,
  CornerUpLeft,
  CornerUpRight,
  ArrowUp,
  Star,
  Trash2,
  AlertCircle,
  SearchX,
  Locate,
} from "lucide-react";

/* ========================================================================== */
/* 1. Types                                                                   */
/* ========================================================================== */

/** The three POI categories supported by the map. */
export type Category = "cafe" | "restaurant" | "park";

/** A point of interest. */
export interface Place {
  id: string;
  name: string;
  category: Category;
  address: string;
  /** [latitude, longitude] */
  coords: LatLngTuple;
  rating: number;
  description: string;
  tags: string[];
}

/** One step in a turn-by-turn route. */
export interface RouteStep {
  id: string;
  instruction: string;
  distanceMeters: number;
  maneuver: "depart" | "straight" | "left" | "right" | "arrive";
  /** Index into the polyline's coordinate array where this step begins. */
  pathIndex: number;
}

/** A non-POI origin, e.g. the user's current position. */
export interface UserPoint {
  id: "user";
  name: string;
  coords: LatLngTuple;
}

/** Result of a route calculation. */
export interface RouteResult {
  origin: Place | UserPoint;
  destination: Place;
  path: LatLngTuple[];
  /** meters */
  totalDistance: number;
  durationMinutes: number;
  steps: RouteStep[];
}

/* ========================================================================== */
/* 2. Mock dataset                                                            */
/* ========================================================================== */

/** Riyadh-centered mock dataset. */
export const PLACES: Place[] = [
  {
    id: "p1",
    name: "Olea Coffee Roasters",
    category: "cafe",
    address: "King Fahd Rd, Al Olaya",
    coords: [24.6908, 46.6853],
    rating: 4.7,
    description: "Single-origin pour-overs and a quiet courtyard seating area.",
    tags: ["specialty", "wifi", "quiet"],
  },
  {
    id: "p2",
    name: "Brew & Bloom",
    category: "cafe",
    address: "Tahlia St, Al Malqa",
    coords: [24.8012, 46.6402],
    rating: 4.4,
    description: "Floral lattes and all-day brunch in a bright, airy space.",
    tags: ["brunch", "pastries"],
  },
  {
    id: "p3",
    name: "Najd Grill House",
    category: "restaurant",
    address: "Prince Turki St, Al Murabba",
    coords: [24.6559, 46.7106],
    rating: 4.6,
    description: "Charcoal-grilled kebabs and traditional Najdi mezze.",
    tags: ["grill", "family", "traditional"],
  },
  {
    id: "p4",
    name: "Sushi Ren",
    category: "restaurant",
    address: "Anas bin Malik Rd, Al Wurud",
    coords: [24.7215, 46.6998],
    rating: 4.3,
    description: "Omakase counter and a curated sake list.",
    tags: ["japanese", "omakase"],
  },
  {
    id: "p5",
    name: "Wadi Hanifah Trail Park",
    category: "park",
    address: "Wadi Hanifah, Western Ring",
    coords: [24.6123, 46.6341],
    rating: 4.8,
    description: "Riverside walking trails, cycling lanes, and shaded picnic zones.",
    tags: ["trail", "outdoor", "cycling"],
  },
  {
    id: "p6",
    name: "King Abdullah Park",
    category: "park",
    address: "King Abdullah Rd, Al Hamra",
    coords: [24.7417, 46.6707],
    rating: 4.5,
    description: "Large urban park with a children's play area and lake views.",
    tags: ["family", "lake", "playground"],
  },
  {
    id: "p7",
    name: "Dir'iyah Cafe Lounge",
    category: "cafe",
    address: "Al Turaif, Dir'iyah",
    coords: [24.7344, 46.5749],
    rating: 4.2,
    description: "Mud-brick courtyard café beside the historic quarter.",
    tags: ["historic", "outdoor"],
  },
  {
    id: "p8",
    name: "Al Dhiyafa Kitchen",
    category: "restaurant",
    address: "Olaya St, Al Sahafa",
    coords: [24.7548, 46.7081],
    rating: 4.1,
    description: "Hearty Levantine dishes served family style.",
    tags: ["levantine", "family"],
  },
];

/** Labels, colors, and icons for each category. */
export const CATEGORY_META: Record<
  Category,
  { label: string; color: string; Icon: FC<{ className?: string }> }
> = {
  cafe: { label: "Cafés", color: "#b45309", Icon: Coffee },
  restaurant: { label: "Restaurants", color: "#be123c", Icon: Utensils },
  park: { label: "Parks", color: "#15803d", Icon: Trees },
};

/** Default map center (central Riyadh) and simulated user location. */
const RIYADH_CENTER: LatLngTuple = [24.7136, 46.6753];

/* ========================================================================== */
/* 3. Geo utilities (pure functions, easy to unit test)                       */
/* ========================================================================== */

const EARTH_RADIUS_M = 6_371_000;
const toRad = (deg: number) => (deg * Math.PI) / 180;

/** Haversine great-circle distance in meters. */
export function haversine(a: LatLngTuple, b: LatLngTuple): number {
  const dLat = toRad(b[0] - a[0]);
  const dLon = toRad(b[1] - a[1]);
  const h =
    Math.sin(dLat / 2) ** 2 +
    Math.cos(toRad(a[0])) * Math.cos(toRad(b[0])) * Math.sin(dLon / 2) ** 2;
  return 2 * EARTH_RADIUS_M * Math.asin(Math.sqrt(h));
}

/** Initial bearing from a → b, in degrees [0, 360). */
export function bearing(a: LatLngTuple, b: LatLngTuple): number {
  const phi1 = toRad(a[0]);
  const phi2 = toRad(b[0]);
  const dLambda = toRad(b[1] - a[1]);
  const y = Math.sin(dLambda) * Math.cos(phi2);
  const x =
    Math.cos(phi1) * Math.sin(phi2) -
    Math.sin(phi1) * Math.cos(phi2) * Math.cos(dLambda);
  return ((Math.atan2(y, x) * 180) / Math.PI + 360) % 360;
}

/** Formats meters as "850 m" or "2.4 km". */
export function formatDistance(meters: number): string {
  return meters < 1000
    ? `${Math.round(meters)} m`
    : `${(meters / 1000).toFixed(1)} km`;
}

/** Formats minutes as "8 min" or "1 h 12 min". */
export function formatDuration(minutes: number): string {
  if (minutes < 60) return `${Math.round(minutes)} min`;
  const h = Math.floor(minutes / 60);
  const m = Math.round(minutes % 60);
  return m ? `${h} h ${m} min` : `${h} h`;
}

/**
 * Produces a stylized path between two points: a gently curved quadratic
 * Bézier sampled at fixed steps, so the line reads as a road-like route
 * rather than a straight "as the crow flies" segment.
 */
export function buildStylizedPath(
  from: LatLngTuple,
  to: LatLngTuple,
  samples = 40,
): LatLngTuple[] {
  const midLat = (from[0] + to[0]) / 2;
  const midLon = (from[1] + to[1]) / 2;
  // Control point offset perpendicular to the chord for a gentle arc.
  const dLon = to[1] - from[1];
  const dLat = to[0] - from[0];
  const control: LatLngTuple = [midLat - dLon * 0.15, midLon + dLat * 0.15];

  const points: LatLngTuple[] = [];
  for (let i = 0; i <= samples; i++) {
    const t = i / samples;
    const u = 1 - t;
    points.push([
      u * u * from[0] + 2 * u * t * control[0] + t * t * to[0],
      u * u * from[1] + 2 * u * t * control[1] + t * t * to[1],
    ]);
  }
  return points;
}

/* ========================================================================== */
/* 4. Route calculator (simulated)                                            */
/* ========================================================================== */

const AVERAGE_SPEED_KMH = 28; // simulated urban driving speed
const MIN_STEP_SPACING_M = 150;

/** Converts a bearing delta into a human maneuver. */
function classifyTurn(delta: number): "straight" | "left" | "right" {
  const normalized = ((delta + 540) % 360) - 180; // → (-180, 180]
  if (Math.abs(normalized) < 20) return "straight";
  return normalized > 0 ? "right" : "left";
}

/** Builds granular turn-by-turn instructions from a sampled path. */
export function buildSteps(
  path: LatLngTuple[],
  destinationName: string,
): RouteStep[] {
  if (path.length < 2) return [];

  const steps: RouteStep[] = [
    {
      id: "s0",
      instruction: "Depart from your starting point",
      distanceMeters: 0,
      maneuver: "depart",
      pathIndex: 0,
    },
  ];

  let accumulated = 0; // meters since the last emitted step
  let lastBearing = bearing(path[0], path[1]);

  for (let i = 1; i < path.length - 1; i++) {
    const segmentBearing = bearing(path[i], path[i + 1]);
    const maneuver = classifyTurn(segmentBearing - lastBearing);
    accumulated += haversine(path[i - 1], path[i]);

    // Only emit a step on meaningful turns, spaced at least MIN_STEP_SPACING_M apart.
    if (maneuver !== "straight" && accumulated >= MIN_STEP_SPACING_M) {
      steps.push({
        id: `s${steps.length}`,
        instruction: maneuver === "left" ? "Turn left" : "Turn right",
        distanceMeters: accumulated,
        maneuver,
        pathIndex: i,
      });
      accumulated = 0;
    }
    lastBearing = segmentBearing;
  }

  steps.push({
    id: `s${steps.length}`,
    instruction: `Arrive at ${destinationName}`,
    distanceMeters: accumulated + haversine(path[path.length - 2], path[path.length - 1]),
    maneuver: "arrive",
    pathIndex: path.length - 1,
  });

  return steps;
}

/**
 * useRouteCalculator
 * Given an origin and destination, computes a stylized path, total distance,
 * a duration estimate, and granular turn-by-turn steps. Cancels any pending
 * calculation when the endpoints change, so stale routes never appear.
 */
export function useRouteCalculator(
  origin: Place | UserPoint | null,
  destination: Place | null,
): { route: RouteResult | null; isCalculating: boolean; error: string | null } {
  const [state, setState] = useState<{
    route: RouteResult | null;
    isCalculating: boolean;
    error: string | null;
  }>({ route: null, isCalculating: false, error: null });

  useEffect(() => {
    if (!origin || !destination) {
      setState({ route: null, isCalculating: false, error: null });
      return;
    }
    if (
      origin.coords[0] === destination.coords[0] &&
      origin.coords[1] === destination.coords[1]
    ) {
      setState({
        route: null,
        isCalculating: false,
        error: "Origin and destination are the same.",
      });
      return;
    }

    setState((s) => ({ ...s, isCalculating: true, error: null }));

    // Simulated latency mirrors a real routing service.
    const timer = window.setTimeout(() => {
      try {
        const path = buildStylizedPath(origin.coords, destination.coords, 48);
        const totalDistance = path.reduce(
          (sum, p, i) => (i === 0 ? 0 : sum + haversine(path[i - 1], p)),
          0,
        );
        const durationMinutes =
          (totalDistance / 1000 / AVERAGE_SPEED_KMH) * 60 + 2;
        const steps = buildSteps(path, destination.name);

        setState({
          route: {
            origin,
            destination,
            path,
            totalDistance,
            durationMinutes,
            steps,
          },
          isCalculating: false,
          error: null,
        });
      } catch (e) {
        setState({
          route: null,
          isCalculating: false,
          error: e instanceof Error ? e.message : "Route calculation failed.",
        });
      }
    }, 450);

    return () => window.clearTimeout(timer);
  }, [origin, destination]);

  return state;
}

/* ========================================================================== */
/* 5. Bookmarks (localStorage-synced)                                         */
/* ========================================================================== */

const BOOKMARK_KEY = "mappulse:favorites:v1";

function readBookmarks(): string[] {
  try {
    const raw = window.localStorage.getItem(BOOKMARK_KEY);
    const parsed: unknown = raw ? JSON.parse(raw) : [];
    return Array.isArray(parsed)
      ? parsed.filter((x): x is string => typeof x === "string")
      : [];
  } catch {
    // Corrupt JSON or storage disabled (private mode) → start empty.
    return [];
  }
}

/**
 * useBookmarks
 * Persists favorite place IDs. Syncs across browser tabs via the `storage`
 * event so a favorite added in one tab appears in the others.
 */
export function useBookmarks() {
  const [ids, setIds] = useState<string[]>(readBookmarks);

  useEffect(() => {
    try {
      window.localStorage.setItem(BOOKMARK_KEY, JSON.stringify(ids));
    } catch {
      /* quota exceeded or storage blocked — fail silently */
    }
  }, [ids]);

  useEffect(() => {
    const onStorage = (e: StorageEvent) => {
      if (e.key === BOOKMARK_KEY) setIds(readBookmarks());
    };
    window.addEventListener("storage", onStorage);
    return () => window.removeEventListener("storage", onStorage);
  }, []);

  const isBookmarked = useCallback((id: string) => ids.includes(id), [ids]);

  const toggle = useCallback(
    (id: string) =>
      setIds((prev) =>
        prev.includes(id) ? prev.filter((x) => x !== id) : [...prev, id],
      ),
    [],
  );

  const remove = useCallback(
    (id: string) => setIds((prev) => prev.filter((x) => x !== id)),
    [],
  );

  return { ids, isBookmarked, toggle, remove };
}

/* ========================================================================== */
/* 6. Debounce hook                                                           */
/* ========================================================================== */

function useDebounced<T>(value: T, delay = 180): T {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const t = window.setTimeout(() => setDebounced(value), delay);
    return () => window.clearTimeout(t);
  }, [value, delay]);
  return debounced;
}

/* ========================================================================== */
/* 7. App state machine                                                       */
/* ========================================================================== */

/**
 * States:
 *  idle          – map browsing, no selection
 *  searching     – search input focused / has a query
 *  viewing_place – a place is selected; detail card shown; camera flown to it
 *  routing       – a route from origin to destination is active
 */
export type AppStatus = "idle" | "searching" | "viewing_place" | "routing";

export type AppState = {
  status: AppStatus;
  query: string;
  category: Category | "all";
  selectedPlace: Place | null;
  origin: Place | UserPoint | null;
  destination: Place | null;
  showFavorites: boolean;
  /** Increments whenever the camera should fly; drives MapController. */
  flyNonce: number;
  flyTarget: LatLngTuple | null;
  flyZoom: number;
};

export type AppAction =
  | { type: "SET_QUERY"; query: string }
  | { type: "SET_CATEGORY"; category: Category | "all" }
  | { type: "FOCUS_SEARCH" }
  | { type: "CLEAR_SEARCH" }
  | { type: "SELECT_PLACE"; place: Place }
  | { type: "CLOSE_PLACE" }
  | { type: "START_ROUTE"; from: Place | UserPoint; to: Place }
  | { type: "END_ROUTE" }
  | { type: "TOGGLE_FAVORITES" }
  | { type: "FLY_TO"; target: LatLngTuple; zoom?: number }
  | { type: "RESET" };

export const initialAppState: AppState = {
  status: "idle",
  query: "",
  category: "all",
  selectedPlace: null,
  origin: null,
  destination: null,
  showFavorites: false,
  flyNonce: 0,
  flyTarget: null,
  flyZoom: 14,
};

/**
 * Pure reducer. Every transition is explicit; invalid transitions are no-ops,
 * which keeps the UI from ever reaching an impossible combination.
 */
export function appReducer(state: AppState, action: AppAction): AppState {
  switch (action.type) {
    case "SET_QUERY": {
      const hasQuery = action.query.trim().length > 0;
      const keepStatus =
        state.status === "routing" || state.status === "viewing_place";
      return {
        ...state,
        query: action.query,
        status: keepStatus ? state.status : hasQuery ? "searching" : "idle",
      };
    }

    case "SET_CATEGORY":
      return { ...state, category: action.category };

    case "FOCUS_SEARCH":
      return state.status === "idle" ? { ...state, status: "searching" } : state;

    case "CLEAR_SEARCH":
      return {
        ...state,
        query: "",
        status: state.status === "searching" ? "idle" : state.status,
      };

    case "SELECT_PLACE":
      return {
        ...state,
        status: "viewing_place",
        selectedPlace: action.place,
        query: action.place.name,
        flyTarget: action.place.coords,
        flyZoom: 16,
        flyNonce: state.flyNonce + 1,
        showFavorites: false,
      };

    case "CLOSE_PLACE":
      if (state.status === "routing") return state;
      return { ...state, status: "idle", selectedPlace: null };

    case "START_ROUTE":
      return {
        ...state,
        status: "routing",
        origin: action.from,
        destination: action.to,
        selectedPlace: action.to,
      };

    case "END_ROUTE":
      return {
        ...state,
        status: state.selectedPlace ? "viewing_place" : "idle",
        origin: null,
        destination: null,
      };

    case "TOGGLE_FAVORITES":
      return { ...state, showFavorites: !state.showFavorites };

    case "FLY_TO":
      return {
        ...state,
        flyTarget: action.target,
        flyZoom: action.zoom ?? 14,
        flyNonce: state.flyNonce + 1,
      };

    case "RESET":
      return initialAppState;

    default:
      return state;
  }
}

/** Convenience hook exposing the reducer. */
export function useAppMachine() {
  const [state, dispatch] = useReducer(appReducer, initialAppState);
  return { state, dispatch };
}

/* ========================================================================== */
/* 8. Search & filter                                                         */
/* ========================================================================== */

export type SearchResult =
  | { kind: "ok"; places: Place[] }
  | { kind: "empty"; query: string }
  | { kind: "error"; message: string };

/**
 * Pure search function. Matches name, address, tags, and description,
 * ranks name-prefix matches first, then applies the category filter.
 * Returns a controlled error result instead of throwing, so the UI can show it.
 */
export function searchPlaces(
  query: string,
  category: Category | "all",
  source: Place[] = PLACES,
): SearchResult {
  try {
    const q = query.trim().toLowerCase();
    const pool =
      category === "all"
        ? source
        : source.filter((p) => p.category === category);

    if (!q) return { kind: "ok", places: pool };

    const scored = pool
      .map((p) => {
        const name = p.name.toLowerCase();
        const haystack = [p.address, p.description, ...p.tags]
          .join(" ")
          .toLowerCase();
        if (name.startsWith(q)) return { p, score: 3 };
        if (name.includes(q)) return { p, score: 2 };
        if (haystack.includes(q)) return { p, score: 1 };
        return null;
      })
      .filter((x): x is { p: Place; score: number } => x !== null)
      .sort((a, b) => b.score - a.score || b.p.rating - a.p.rating);

    if (scored.length === 0) return { kind: "empty", query };
    return { kind: "ok", places: scored.map((s) => s.p) };
  } catch (e) {
    return {
      kind: "error",
      message: e instanceof Error ? e.message : "Search failed unexpectedly.",
    };
  }
}

/* ========================================================================== */
/* 9. Map components                                                          */
/* ========================================================================== */

/** Inline glyph paths (Lucide-equivalent) sized for a 16px box. */
function categoryGlyphSvg(category: Category): string {
  switch (category) {
    case "cafe":
      return `<path d="M3 8h11v5a4 4 0 0 1-4 4H7a4 4 0 0 1-4-4z"/><path d="M14 9h2a2 2 0 0 1 0 4h-2"/>`;
    case "restaurant":
      return `<path d="M4 2v6a2 2 0 0 0 2 2h0a2 2 0 0 0 2-2V2"/><path d="M6 10v12"/><path d="M14 2c-1.5 1.5-2 4-2 7h2v13"/>`;
    case "park":
      return `<path d="M12 2l5 8h-3l4 6H6l4-6H7z"/><path d="M12 16v6"/>`;
  }
}

/** Builds a Leaflet divIcon containing an inline SVG pin for a category. */
export function createCategoryIcon(
  category: Category,
  highlighted = false,
): L.DivIcon {
  const { color } = CATEGORY_META[category];
  const scale = highlighted ? 1.25 : 1;
  const width = 34 * scale;
  const height = 42 * scale;
  const glyph = categoryGlyphSvg(category);
  const html = `
    <div style="filter: drop-shadow(0 2px 3px rgba(0,0,0,.35)); transform: translate(-50%, -100%);">
      <svg viewBox="0 0 34 42" width="${width}" height="${height}" xmlns="http://www.w3.org/2000/svg">
        <path d="M17 0C7.6 0 0 7.6 0 17c0 12.5 17 25 17 25s17-12.5 17-25C34 7.6 26.4 0 17 0z"
              fill="${color}" stroke="white" stroke-width="2"/>
        <g transform="translate(9,8)" fill="none" stroke="white" stroke-width="1.8"
           stroke-linecap="round" stroke-linejoin="round">${glyph}</g>
      </svg>
    </div>`;
  return L.divIcon({
    html,
    className: "mappulse-marker",
    iconSize: [0, 0],
    iconAnchor: [0, 0],
  });
}

/** Blue dot used for the route origin. */
function originIcon(): L.DivIcon {
  return L.divIcon({
    className: "",
    html: `<div style="width:16px;height:16px;border-radius:9999px;background:#2563eb;border:3px solid white;box-shadow:0 0 0 4px rgba(37,99,235,.25)"></div>`,
    iconSize: [16, 16],
    iconAnchor: [8, 8],
  });
}

/** Imperative camera controller — flies the map whenever flyNonce changes. */
function MapController({
  flyNonce,
  flyTarget,
  flyZoom,
  routePath,
}: {
  flyNonce: number;
  flyTarget: LatLngTuple | null;
  flyZoom: number;
  routePath: LatLngTuple[] | null;
}) {
  const map = useMap();
  const lastNonce = useRef(-1);

  // Frame the full route whenever a new route path arrives.
  useEffect(() => {
    if (routePath && routePath.length > 1) {
      map.flyToBounds(L.latLngBounds(routePath), {
        padding: [60, 60],
        duration: 1.1,
      });
    }
  }, [map, routePath]);

  // Fly to a target whenever the nonce changes (selecting the same place
  // twice still re-flies the camera).
  useEffect(() => {
    if (flyTarget && flyNonce !== lastNonce.current) {
      lastNonce.current = flyNonce;
      map.flyTo(flyTarget, flyZoom, { duration: 1.2, easeLinearity: 0.25 });
    }
  }, [map, flyNonce, flyTarget, flyZoom]);

  return null;
}

/** Keeps the Leaflet canvas sized correctly when the container resizes. */
function ResizeHandler() {
  const map = useMap();
  useEffect(() => {
    const el = map.getContainer();
    const ro = new ResizeObserver(() => map.invalidateSize({ animate: false }));
    ro.observe(el);
    return () => ro.disconnect();
  }, [map]);
  return null;
}

/** Reports map clicks to the parent, used to dismiss transient UI. */
function ClickDismiss({ onClick }: { onClick: () => void }) {
  useMapEvents({ click: onClick });
  return null;
}

interface MapViewProps {
  places: Place[];
  selectedId: string | null;
  route: RouteResult | null;
  flyNonce: number;
  flyTarget: LatLngTuple | null;
  flyZoom: number;
  onSelect: (p: Place) => void;
  onMapClick: () => void;
}

/** The interactive Leaflet canvas. */
export const MapView: FC<MapViewProps> = ({
  places,
  selectedId,
  route,
  flyNonce,
  flyTarget,
  flyZoom,
  onSelect,
  onMapClick,
}) => {
  const center: LatLngExpression = RIYADH_CENTER;

  return (
    <MapContainer
      center={center}
      zoom={12}
      className="h-full w-full"
      zoomControl={false}
      attributionControl
    >
      <TileLayer
        url="https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png"
        attribution='&copy; <a href="https://www.openstreetmap.org/copyright">OSM</a> &copy; <a href="https://carto.com/attributions">CARTO</a>'
      />
      <MapController
        flyNonce={flyNonce}
        flyTarget={flyTarget}
        flyZoom={flyZoom}
        routePath={route ? route.path : null}
      />
      <ResizeHandler />
      <ClickDismiss onClick={onMapClick} />

      {places.map((p) => (
        <Marker
          key={p.id}
          position={p.coords}
          icon={createCategoryIcon(p.category, selectedId === p.id)}
          eventHandlers={{ click: () => onSelect(p) }}
        >
          <Popup>
            <div className="text-sm font-medium">{p.name}</div>
          </Popup>
        </Marker>
      ))}

      {route && (
        <>
          <Polyline
            positions={route.path}
            pathOptions={{
              color: "#ffffff",
              weight: 9,
              opacity: 0.9,
              lineCap: "round",
            }}
          />
          <Polyline
            positions={route.path}
            pathOptions={{
              color: "#2563eb",
              weight: 5,
              opacity: 1,
              lineCap: "round",
            }}
          />
          <Marker position={route.origin.coords} icon={originIcon()} />
        </>
      )}
    </MapContainer>
  );
};

/* ========================================================================== */
/* 10. Sidebar components                                                     */
/* ========================================================================== */

/** Small colored dot indicating a place's category. */
const CategoryDot: FC<{ category: Category }> = ({ category }) => {
  const { color, Icon } = CATEGORY_META[category];
  return (
    <span
      className="flex h-7 w-7 shrink-0 items-center justify-center rounded-full text-white"
      style={{ backgroundColor: color }}
    >
      <Icon className="h-3.5 w-3.5" />
    </span>
  );
};

/** Auto-suggest search input with keyboard navigation. */
export const SearchBar: FC<{
  value: string;
  onChange: (v: string) => void;
  onFocus: () => void;
  onClear: () => void;
  suggestions: Place[];
  onPick: (p: Place) => void;
}> = ({ value, onChange, onFocus, onClear, suggestions, onPick }) => {
  const [open, setOpen] = useState(false);
  const [active, setActive] = useState(0);
  const items = suggestions.slice(0, 6);
  const showList = open && value.trim().length > 0 && items.length > 0;

  const handleKey = (e: KeyboardEvent<HTMLInputElement>) => {
    if (!showList) return;
    if (e.key === "ArrowDown") {
      e.preventDefault();
      setActive((i) => (i + 1) % items.length);
    } else if (e.key === "ArrowUp") {
      e.preventDefault();
      setActive((i) => (i - 1 + items.length) % items.length);
    } else if (e.key === "Enter") {
      e.preventDefault();
      onPick(items[active]);
      setOpen(false);
    } else if (e.key === "Escape") {
      setOpen(false);
    }
  };

  return (
    <div className="relative">
      <div className="flex items-center gap-2 rounded-xl bg-white px-3 py-2.5 shadow-sm ring-1 ring-slate-200 focus-within:ring-2 focus-within:ring-blue-500">
        <Search className="h-4 w-4 shrink-0 text-slate-400" aria-hidden />
        <input
          role="combobox"
          aria-expanded={showList}
          aria-autocomplete="list"
          aria-controls="search-suggestions"
          value={value}
          onChange={(e) => {
            onChange(e.target.value);
            setOpen(true);
            setActive(0);
          }}
          onFocus={() => {
            setOpen(true);
            onFocus();
          }}
          onBlur={() => window.setTimeout(() => setOpen(false), 120)}
          onKeyDown={handleKey}
          placeholder="Search places in Riyadh"
          className="min-w-0 flex-1 bg-transparent text-sm text-slate-900 outline-none placeholder:text-slate-400"
        />
        {value && (
          <button
            type="button"
            onClick={() => {
              onClear();
              setOpen(false);
            }}
            aria-label="Clear search"
            className="rounded-full p-1 text-slate-400 hover:bg-slate-100 hover:text-slate-600"
          >
            <X className="h-3.5 w-3.5" />
          </button>
        )}
      </div>

      {showList && (
        <ul
          id="search-suggestions"
          role="listbox"
          className="absolute left-0 right-0 z-30 mt-2 max-h-72 overflow-y-auto rounded-xl bg-white py-1 shadow-lg ring-1 ring-slate-200"
        >
          {items.map((p, i) => (
            <li
              key={p.id}
              role="option"
              aria-selected={i === active}
              onMouseDown={(e) => {
                e.preventDefault();
                onPick(p);
                setOpen(false);
              }}
              onMouseEnter={() => setActive(i)}
              className={`flex cursor-pointer items-center gap-3 px-3 py-2 text-sm ${
                i === active ? "bg-blue-50" : "hover:bg-slate-50"
              }`}
            >
              <CategoryDot category={p.category} />
              <div className="min-w-0">
                <div className="truncate font-medium text-slate-900">
                  {p.name}
                </div>
                <div className="truncate text-xs text-slate-500">
                  {p.address}
                </div>
              </div>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
};

/** Top-of-map overlay for quick category filtering. */
export const CategoryOverlay: FC<{
  active: Category | "all";
  onChange: (c: Category | "all") => void;
  counts: Record<Category | "all", number>;
}> = ({ active, onChange, counts }) => {
  const options: Array<{
    value: Category | "all";
    label: string;
    Icon?: FC<{ className?: string }>;
  }> = [
    { value: "all", label: "All" },
    { value: "cafe", label: CATEGORY_META.cafe.label, Icon: CATEGORY_META.cafe.Icon },
    {
      value: "restaurant",
      label: CATEGORY_META.restaurant.label,
      Icon: CATEGORY_META.restaurant.Icon,
    },
    { value: "park", label: CATEGORY_META.park.label, Icon: CATEGORY_META.park.Icon },
  ];

  return (
    <div
      role="toolbar"
      aria-label="Filter places by category"
      className="absolute left-1/2 top-4 z-[500] flex -translate-x-1/2 gap-1.5 rounded-full bg-white/95 p-1.5 shadow-lg ring-1 ring-slate-200 backdrop-blur"
    >
      {options.map(({ value, label, Icon }) => {
        const isActive = active === value;
        return (
          <button
            key={value}
            type="button"
            aria-pressed={isActive}
            onClick={() => onChange(value)}
            className={`flex items-center gap-1.5 whitespace-nowrap rounded-full px-3 py-1.5 text-xs font-medium transition ${
              isActive ? "bg-slate-900 text-white" : "text-slate-700 hover:bg-slate-100"
            }`}
          >
            {Icon && <Icon className="h-3.5 w-3.5" />}
            {label}
            <span
              className={`rounded-full px-1.5 text-[10px] ${
                isActive ? "bg-white/20" : "bg-slate-100 text-slate-500"
              }`}
            >
              {counts[value]}
            </span>
          </button>
        );
      })}
    </div>
  );
};

const StateCard: FC<{
  tone: "error" | "empty";
  icon: ReactNode;
  title: string;
  body: string;
  action?: { label: string; onClick: () => void };
}> = ({ tone, icon, title, body, action }) => (
  <div
    role={tone === "error" ? "alert" : "status"}
    className={`flex flex-col items-center gap-2 rounded-xl border px-4 py-6 text-center ${
      tone === "error"
        ? "border-rose-200 bg-rose-50 text-rose-700"
        : "border-slate-200 bg-slate-50 text-slate-600"
    }`}
  >
    {icon}
    <div className="text-sm font-semibold">{title}</div>
    <p className="max-w-[240px] text-xs leading-relaxed opacity-80">{body}</p>
    {action && (
      <button
        type="button"
        onClick={action.onClick}
        className="mt-1 rounded-lg bg-white px-3 py-1.5 text-xs font-medium text-slate-700 shadow-sm ring-1 ring-slate-200 hover:bg-slate-50"
      >
        {action.label}
      </button>
    )}
  </div>
);

/** Search status: shown for error or no-results. */
export const SearchStatus: FC<{ result: SearchResult; onClear: () => void }> = ({
  result,
  onClear,
}) => {
  if (result.kind === "ok") return null;

  if (result.kind === "error") {
    return (
      <StateCard
        tone="error"
        icon={<AlertCircle className="h-6 w-6" />}
        title="Something went wrong"
        body={result.message}
        action={{ label: "Try again", onClick: onClear }}
      />
    );
  }

  return (
    <StateCard
      tone="empty"
      icon={<SearchX className="h-6 w-6" />}
      title="No places found"
      body={`Nothing matches “${result.query}”. Try a shorter term or a different category.`}
      action={{ label: "Clear search", onClick: onClear }}
    />
  );
};

/** Selected-place detail card with bookmark and routing actions. */
export const PlaceCard: FC<{
  place: Place;
  isBookmarked: boolean;
  onToggleBookmark: () => void;
  onRoute: () => void;
  onClose: () => void;
}> = ({ place, isBookmarked, onToggleBookmark, onRoute, onClose }) => {
  const { label, color, Icon } = CATEGORY_META[place.category];
  return (
    <article className="space-y-3 rounded-xl bg-white p-4 shadow-sm ring-1 ring-slate-200">
      <header className="flex items-start gap-3">
        <span
          className="flex h-10 w-10 shrink-0 items-center justify-center rounded-lg text-white"
          style={{ backgroundColor: color }}
        >
          <Icon className="h-5 w-5" />
        </span>
        <div className="min-w-0 flex-1">
          <h2 className="truncate text-base font-semibold text-slate-900">
            {place.name}
          </h2>
          <p className="flex items-center gap-1 text-xs text-slate-500">
            <span className="font-medium text-slate-700">{label}</span>
            <span>·</span>
            <Star className="h-3 w-3 fill-amber-400 text-amber-400" />
            {place.rating.toFixed(1)}
          </p>
        </div>
        <button
          type="button"
          onClick={onClose}
          aria-label="Close details"
          className="rounded-full p-1 text-slate-400 hover:bg-slate-100"
        >
          <X className="h-4 w-4" />
        </button>
      </header>

      <p className="flex items-start gap-1.5 text-xs text-slate-600">
        <MapPin className="mt-0.5 h-3.5 w-3.5 shrink-0" />
        {place.address}
      </p>
      <p className="text-sm leading-relaxed text-slate-700">{place.description}</p>

      <div className="flex flex-wrap gap-1.5">
        {place.tags.map((t) => (
          <span
            key={t}
            className="rounded-full bg-slate-100 px-2 py-0.5 text-[11px] text-slate-600"
          >
            {t}
          </span>
        ))}
      </div>

      <div className="flex gap-2 pt-1">
        <button
          type="button"
          onClick={onRoute}
          className="flex flex-1 items-center justify-center gap-1.5 rounded-lg bg-blue-600 px-3 py-2 text-sm font-medium text-white hover:bg-blue-700"
        >
          <Navigation className="h-4 w-4" />
          Directions
        </button>
        <button
          type="button"
          onClick={onToggleBookmark}
          aria-pressed={isBookmarked}
          className={`flex items-center justify-center gap-1.5 rounded-lg px-3 py-2 text-sm font-medium ring-1 ${
            isBookmarked
              ? "bg-amber-50 text-amber-700 ring-amber-200 hover:bg-amber-100"
              : "bg-white text-slate-700 ring-slate-200 hover:bg-slate-50"
          }`}
        >
          {isBookmarked ? (
            <BookmarkCheck className="h-4 w-4" />
          ) : (
            <Bookmark className="h-4 w-4" />
          )}
          {isBookmarked ? "Saved" : "Save"}
        </button>
      </div>
    </article>
  );
};

const MANEUVER_ICON: Record<RouteStep["maneuver"], FC<{ className?: string }>> = {
  depart: ArrowUp,
  straight: ArrowUp,
  left: CornerUpLeft,
  right: CornerUpRight,
  arrive: MapPin,
};

const Stat: FC<{ icon: ReactNode; label: string; value: string }> = ({
  icon,
  label,
  value,
}) => (
  <div className="flex items-center gap-2 rounded-lg bg-slate-50 px-3 py-2">
    <span className="text-slate-500">{icon}</span>
    <div>
      <div className="text-[11px] uppercase tracking-wide text-slate-500">
        {label}
      </div>
      <div className="text-sm font-semibold text-slate-900">{value}</div>
    </div>
  </div>
);

/** Route summary and turn-by-turn list. */
export const RoutePanel: FC<{
  route: RouteResult | null;
  isCalculating: boolean;
  error: string | null;
  onBack: () => void;
  onExit: () => void;
}> = ({ route, isCalculating, error, onBack, onExit }) => {
  if (isCalculating) {
    return (
      <div
        className="space-y-3 rounded-xl bg-white p-4 shadow-sm ring-1 ring-slate-200"
        aria-busy="true"
      >
        <div className="h-4 w-1/2 animate-pulse rounded bg-slate-200" />
        <div className="h-3 w-3/4 animate-pulse rounded bg-slate-100" />
        <div className="h-3 w-2/3 animate-pulse rounded bg-slate-100" />
      </div>
    );
  }

  if (error) {
    return (
      <StateCard
        tone="error"
        icon={<AlertCircle className="h-6 w-6" />}
        title="Route unavailable"
        body={error}
        action={{ label: "Go back", onClick: onBack }}
      />
    );
  }

  if (!route) return null;

  return (
    <section className="space-y-3 rounded-xl bg-white p-4 shadow-sm ring-1 ring-slate-200">
      <header className="flex items-center gap-2">
        <button
          type="button"
          onClick={onBack}
          aria-label="Back to place"
          className="rounded-full p-1 text-slate-500 hover:bg-slate-100"
        >
          <ArrowLeft className="h-4 w-4" />
        </button>
        <div className="min-w-0 flex-1">
          <div className="truncate text-xs text-slate-500">To</div>
          <div className="truncate text-sm font-semibold text-slate-900">
            {route.destination.name}
          </div>
        </div>
        <button
          type="button"
          onClick={onExit}
          className="rounded-lg px-2 py-1 text-xs font-medium text-rose-600 hover:bg-rose-50"
        >
          Exit
        </button>
      </header>

      <div className="grid grid-cols-2 gap-2">
        <Stat
          icon={<Clock className="h-4 w-4" />}
          label="Duration"
          value={formatDuration(route.durationMinutes)}
        />
        <Stat
          icon={<Ruler className="h-4 w-4" />}
          label="Distance"
          value={formatDistance(route.totalDistance)}
        />
      </div>

      <ol className="max-h-[38vh] space-y-1 overflow-y-auto pr-1">
        {route.steps.map((step) => {
          const Icon = MANEUVER_ICON[step.maneuver];
          return (
            <li
              key={step.id}
              className="flex items-start gap-3 rounded-lg px-2 py-2 hover:bg-slate-50"
            >
              <span className="mt-0.5 flex h-6 w-6 shrink-0 items-center justify-center rounded-full bg-blue-50 text-blue-600">
                <Icon className="h-3.5 w-3.5" />
              </span>
              <div className="min-w-0 flex-1">
                <div className="text-sm text-slate-800">{step.instruction}</div>
                {step.maneuver !== "depart" && step.maneuver !== "arrive" && (
                  <div className="text-xs text-slate-500">
                    {formatDistance(step.distanceMeters)} ahead
                  </div>
                )}
              </div>
            </li>
          );
        })}
      </ol>
    </section>
  );
};

/** Favorites list with remove actions and empty state. */
export const FavoritesList: FC<{
  places: Place[];
  onSelect: (p: Place) => void;
  onRemove: (id: string) => void;
}> = ({ places, onSelect, onRemove }) => {
  if (places.length === 0) {
    return (
      <StateCard
        tone="empty"
        icon={<Bookmark className="h-6 w-6" />}
        title="No favorites yet"
        body="Save a place from its details card and it will appear here."
      />
    );
  }
  return (
    <ul className="space-y-2">
      {places.map((p) => (
        <li
          key={p.id}
          className="group flex items-center gap-3 rounded-xl bg-white p-2.5 shadow-sm ring-1 ring-slate-200 hover:ring-blue-300"
        >
          <button
            type="button"
            onClick={() => onSelect(p)}
            className="flex min-w-0 flex-1 items-center gap-3 text-left"
          >
            <CategoryDot category={p.category} />
            <div className="min-w-0">
              <div className="truncate text-sm font-medium text-slate-900">
                {p.name}
              </div>
              <div className="truncate text-xs text-slate-500">{p.address}</div>
            </div>
          </button>
          <button
            type="button"
            onClick={() => onRemove(p.id)}
            aria-label={`Remove ${p.name} from favorites`}
            className="rounded-full p-1.5 text-slate-400 opacity-0 transition hover:bg-rose-50 hover:text-rose-600 focus:opacity-100 group-hover:opacity-100"
          >
            <Trash2 className="h-4 w-4" />
          </button>
        </li>
      ))}
    </ul>
  );
};

/* ========================================================================== */
/* 11. Root application                                                       */
/* ========================================================================== */

export default function MapPulse() {
  const { state, dispatch } = useAppMachine();
  const bookmarks = useBookmarks();
  const debouncedQuery = useDebounced(state.query, 180);

  const searchResult = useMemo(
    () => searchPlaces(debouncedQuery, state.category),
    [debouncedQuery, state.category],
  );

  const visiblePlaces = searchResult.kind === "ok" ? searchResult.places : [];

  const counts = useMemo(() => {
    const base: Record<Category | "all", number> = {
      all: PLACES.length,
      cafe: 0,
      restaurant: 0,
      park: 0,
    };
    for (const p of PLACES) base[p.category]++;
    return base;
  }, []);

  const favoritePlaces = useMemo(
    () =>
      bookmarks.ids
        .map((id) => PLACES.find((p) => p.id === id))
        .filter((p): p is Place => !!p),
    [bookmarks.ids],
  );

  const userOrigin: UserPoint = useMemo(
    () => ({ id: "user", name: "Your location", coords: RIYADH_CENTER }),
    [],
  );

  const routeOrigin = state.status === "routing" ? (state.origin ?? userOrigin) : null;
  const routeDestination = state.status === "routing" ? state.destination : null;
  const {
    route,
    isCalculating,
    error: routeError,
  } = useRouteCalculator(routeOrigin, routeDestination);

  const handleSelect = useCallback(
    (place: Place) => dispatch({ type: "SELECT_PLACE", place }),
    [dispatch],
  );

  const handleRecenter = useCallback(
    () => dispatch({ type: "FLY_TO", target: RIYADH_CENTER, zoom: 13 }),
    [dispatch],
  );

  return (
    <div className="flex h-screen w-full overflow-hidden bg-slate-100 font-sans">
      {/* ---------- Sidebar: fixed 360px on md+, full-width sheet on mobile ---------- */}
      <aside className="z-[600] flex h-full w-full shrink-0 flex-col border-r border-slate-200 bg-slate-50 md:w-[360px]">
        <header className="space-y-3 border-b border-slate-200 bg-white p-4">
          <div className="flex items-center gap-2">
            <span className="flex h-8 w-8 items-center justify-center rounded-lg bg-slate-900 text-white">
              <MapPin className="h-4 w-4" />
            </span>
            <h1 className="text-lg font-bold tracking-tight text-slate-900">
              MapPulse
            </h1>
            <button
              type="button"
              onClick={() => dispatch({ type: "TOGGLE_FAVORITES" })}
              aria-pressed={state.showFavorites}
              className={`ml-auto flex items-center gap-1.5 rounded-lg px-2.5 py-1.5 text-xs font-medium ring-1 ${
                state.showFavorites
                  ? "bg-amber-50 text-amber-700 ring-amber-200"
                  : "bg-white text-slate-600 ring-slate-200 hover:bg-slate-50"
              }`}
            >
              <Bookmark className="h-3.5 w-3.5" />
              Saved ({bookmarks.ids.length})
            </button>
          </div>

          <SearchBar
            value={state.query}
            onChange={(q) => dispatch({ type: "SET_QUERY", query: q })}
            onFocus={() => dispatch({ type: "FOCUS_SEARCH" })}
            onClear={() => dispatch({ type: "CLEAR_SEARCH" })}
            suggestions={visiblePlaces}
            onPick={handleSelect}
          />
        </header>

        <div className="flex-1 space-y-3 overflow-y-auto p-4">
          {state.showFavorites ? (
            <>
              <h2 className="text-xs font-semibold uppercase tracking-wide text-slate-500">
                Favorite places
              </h2>
              <FavoritesList
                places={favoritePlaces}
                onSelect={handleSelect}
                onRemove={bookmarks.remove}
              />
            </>
          ) : state.status === "routing" ? (
            <RoutePanel
              route={route}
              isCalculating={isCalculating}
              error={routeError}
              onBack={() => dispatch({ type: "END_ROUTE" })}
              onExit={() => dispatch({ type: "END_ROUTE" })}
            />
          ) : state.status === "viewing_place" && state.selectedPlace ? (
            <PlaceCard
              place={state.selectedPlace}
              isBookmarked={bookmarks.isBookmarked(state.selectedPlace.id)}
              onToggleBookmark={() => bookmarks.toggle(state.selectedPlace!.id)}
              onRoute={() =>
                dispatch({
                  type: "START_ROUTE",
                  from: userOrigin,
                  to: state.selectedPlace!,
                })
              }
              onClose={() => dispatch({ type: "CLOSE_PLACE" })}
            />
          ) : (
            <>
              <SearchStatus
                result={searchResult}
                onClear={() => dispatch({ type: "CLEAR_SEARCH" })}
              />
              {searchResult.kind === "ok" && (
                <>
                  <h2 className="text-xs font-semibold uppercase tracking-wide text-slate-500">
                    {state.query.trim() ? "Results" : "Nearby"} · {visiblePlaces.length}
                  </h2>
                  <ul className="space-y-2">
                    {visiblePlaces.map((p) => (
                      <li key={p.id}>
                        <button
                          type="button"
                          onClick={() => handleSelect(p)}
                          className="flex w-full items-center gap-3 rounded-xl bg-white p-2.5 text-left shadow-sm ring-1 ring-slate-200 hover:ring-blue-300"
                        >
                          <CategoryDot category={p.category} />
                          <div className="min-w-0 flex-1">
                            <div className="truncate text-sm font-medium text-slate-900">
                              {p.name}
                            </div>
                            <div className="truncate text-xs text-slate-500">
                              {p.address}
                            </div>
                          </div>
                          <span className="flex items-center gap-0.5 text-xs text-slate-500">
                            <Star className="h-3 w-3 fill-amber-400 text-amber-400" />
                            {p.rating.toFixed(1)}
                          </span>
                        </button>
                      </li>
                    ))}
                  </ul>
                </>
              )}
            </>
          )}
        </div>
      </aside>

      {/* ---------- Map canvas ---------- */}
      <main className="relative hidden flex-1 md:block">
        <CategoryOverlay
          active={state.category}
          onChange={(c) => dispatch({ type: "SET_CATEGORY", category: c })}
          counts={counts}
        />
        <MapView
          places={visiblePlaces}
          selectedId={state.selectedPlace?.id ?? null}
          route={state.status === "routing" ? route : null}
          flyNonce={state.flyNonce}
          flyTarget={state.flyTarget}
          flyZoom={state.flyZoom}
          onSelect={handleSelect}
          onMapClick={() => {
            if (state.status === "searching") dispatch({ type: "CLEAR_SEARCH" });
          }}
        />
        <button
          type="button"
          aria-label="Recenter on your location"
          onClick={handleRecenter}
          className="absolute bottom-6 right-6 z-[500] rounded-full bg-white p-3 text-slate-700 shadow-lg ring-1 ring-slate-200 hover:bg-slate-50"
        >
          <Locate className="h-5 w-5" />
        </button>
      </main>
    </div>
  );
}
