# Project
from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy_garden.mapview import MapView, MapMarker
from plyer import gps
from kivy.clock import Clock
from kivy.properties import NumericProperty, StringProperty
from kivy.lang import Builder

# KV файл для интерфейса
Builder.load_string('''
<LocationApp>:
    orientation: 'vertical'
    MapView:
        id: mapview
        lat: root.latitude
        lon: root.longitude
        zoom: 5
        on_lat: root.update_marker()
        on_lon: root.update_marker()
    Label:
        text: root.location_text
        size_hint_y: 0.1
''')

class LocationApp(BoxLayout):
    latitude = NumericProperty(0)
    longitude = NumericProperty(0)
    location_text = StringProperty('Waiting for GPS...')
    marker = None

    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.mapview = self.ids.mapview
        try:
            gps.configure(on_location=self.on_location, on_status=self.on_status)
            gps.start(1000, 0)  # Обновление каждые 1000 мс
        except NotImplementedError:
            self.location_text = "GPS not supported on this device"
        self.update_marker()

    def on_location(self, **kwargs):
        self.latitude = kwargs.get('lat', 0)
        self.longitude = kwargs.get('lon', 0)
        self.location_text = f"Lat: {self.latitude}, Lon: {self.longitude}"
        self.mapview.center_on(self.latitude, self.longitude)

    def on_status(self, stype, status):
        self.location_text = f"GPS Status: {status}"

    def update_marker(self):
        if self.marker:
            self.mapview.remove_marker(self.marker)
        self.marker = MapMarker(lat=self.latitude, lon=self.longitude)
        self.mapview.add_marker(self.marker)

class GPSMapApp(App):
    def build(self):
        return LocationApp()

if __name__ == '__main__':
    GPSMapApp().run()



