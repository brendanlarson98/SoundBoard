# SoundBoard
## This project creates a SoundBoard class which is designed to create or load a .wav file and perform operations on it, such as reversing it, slowing/speeding it, or even turn peak frequencies into colors.

### Our imports
    import matplotlib.pyplot as plt
    from numpy import fft
    import numpy as np
    from scipy import signal
    from scipy.io import wavfile
    from scipy.io.wavfile import read,write
    import sounddevice as sd
    from numpy.fft import rfft, rfftfreq
    import soundfile as sofi
    from scipy.signal import butter, filtfilt, find_peaks, argrelmax
    
### Here we set out a function, passes, that take in the type of filter (btype = 'high' or 'low') and the cutoff frequency for that is chosen, then returns the new data of a high/low pass filter.
    def passes(data, cutoff, fs, N, btype):
        nyquist = 0.5 * fs
        wn = cutoff / nyquist
        b, a = butter(N, wn, btype, analog=False)
        y = filtfilt(b, a, data)
        return y

### This function, freq_to_color takes in frequency, then moves it into the frequency range of the visible spectrum. We then return the decimal rgb of the color.
    def freq_to_color(frequency):
        e = (10**12)
        while frequency <= 400 * e:
            frequency *= 2
            frequency_list = []
            frequency_list.append(frequency)
            fr = frequency_list[-1]
            if 670 * e <= fr <= 800 * e:
                return (238,130,238)
            elif  620 * e <= fr < 670 * e:
                return (0,0,255)
            elif 600 * e <= fr < 620 * e:
                return (0,255,255)
            elif 530 * e <= fr < 600 * e:
                return (0,255,0)
            elif 510 * e <= fr < 530 * e:
                return (255,255,0)
            elif 480 * e <= fr < 510 * e:
                return (255,165,0)
            elif 400 * e <= fr < 480 * e:
                return (255,0,0)
                
### Our main SoundBoard class, designed to create a soundfile and perform operations on it. We have a standard sampling rate of 44100 Hz.
    class SoundBoard:
        def __init__(self, fs=44100, channels=1, time=4):
            self.channels = channels
            self.time = time
            self.fs = fs
            times = np.linspace(0,self.time, int(self.time * self.fs))
            self.data = 0
            self.freq = 0
            self.fft_lim = 2000
        
#### The load function will load the data which is on a wavfile.   
    def load(self, filepath = "mao.wav"):
        self.data, self.freq = sofi.read(filepath)
        
#### In create, we record a piece of data and then write the data to our path.        
    def create(self, path = "mao.wav"):
        self.data = sd.rec(self.time *self.fs, channels=1, samplerate = self.fs, blocking=True)[:,0]
        write(path, self.fs, self.data)
        
#### The save function writes over the data in the wavefile with whatever the data is now
    def save(self, path="mao.wav"):
        write(path, self.fs, self.data)
        
#### The play function plays the data in the wavfile from your speakers.        
    def play(self):
         sd.play(self.data, self.fs, blocking=True)
            
#### The fft function performes a fast fourier transform for the real values of the data, moving the data into the frequency domain. It returns the associated frequencies and the absolute value of the fft.
    def fft(self):
        fast_ = rfft(self.data)
        absolute_ = np.abs(fast_)
        __freq__ = rfftfreq(len(self.data), d=1/self.fs)
        return __freq__, absolute_
    
#### get_time returns the times for each sample   
    def get_time(self):
        times = np.linspace(0, len(self.data)/self.fs, len(self.data))
        return times
    
#### rev_play plays the data recorded backwards.       
    def rev_play(self):
        self.data=self.data[::-1]
        sd.play(self.data, self.fs, blocking = True)
        
#### reate_play plays the soundfile recorded slower (n = 0.1-0.9) or faster (n=1.1 or larger) than the original data
    def rate_play(self, n):
        sd.play(self.data, n * self.fs, blocking = True)
        
#### graph_plot plots the amplitude of the data versus time.
    def graph_plot(self):
        plt.plot(self.data, 'r', label="Voltage vs. Time")
        plt.legend()
        
#### graph_fft graphs the fast fourier transform of the data in the frequency domain
    def graph_fft(self):
        __freq__, absolute_ = Sound_Record.fft(self)
        plt.plot(__freq__, absolute_, 'b', label="FFT of a Sound File")
        plt.xlim(0,self.fft_lim)
        plt.legend()
        
#### filter_pass plots either a high pass (btype='high') or low pass (btype='low') for its corresponding cutoff point in the frequency domain.
    def filter_pass(self, cutoff, btype):
        N = 6
        __freq__, absolute_ = Sound_Record.fft(self)
        new_plot = passes(absolute_, cutoff, self.fs, N, btype)
        plt.plot(abs(new_plot), 'c', label=f'Plot of {btype.title()} Pass Filter')
        plt.xlim(0,self.fft_lim)
        plt.legend()
        
#### spectrograph plots a spectrograph of the fast fourier transform of the data.
    def spectrograph(self):
        __freq__, absolute_ = Sound_Record.fft(self)
        plt.specgram(absolute_, Fs=self.fs)
        plt.xlabel('Time (s)')
        plt.ylabel('Frequency (Hz)')
        
#### Peaks identifies the peak frequencies in the FFT data from the xlimit of the fft graph and 1000, so in the range of where our strongest data should be (please modify as you see fit for your data). It stores the peak frequencies in a list to be used later. 
    def peaks(self):
        __freq__, absolute_ = Sound_Record.fft(self)
        woot = argrelmax(absolute_, order=60)[0]
        for val in woot:
            if val > self.fft_lim or val < 1000:
                ind = np.where(woot == val)
                woot = np.delete(woot, ind)
            else:
                pass
        return woot
    
#### color takes in the frequencies from the peaks function, and then calls the freq_to_color function to turn those frequencies into colors.
    def color(self):
        lis = Sound_Record.peaks(self)
        for item in lis:
            frequency = item
            colors = freq_to_color(frequency)
            print(f'Frequency: {frequency} Hz')
            plt.imshow([[colors]])
            plt.show()
