//@Tali Pivots are taken from Tradingview Default app and modified the calculation to my own .... 

//@version=5
indicator('[TT] Pivot]', 'Key Levels', overlay=true, max_lines_count=500, max_labels_count=500, max_bars_back=5000)
AUTO = 'Auto'
HOURLY = 'Hourly'
DAILY = 'Daily'
WEEKLY = 'Weekly'
MONTHLY = 'Monthly'
QUARTERLY = 'Quarterly'
YEARLY = 'Yearly'
BIYEARLY = 'Biyearly'
TRIYEARLY = 'Triyearly'
QUINQUENNIAL = 'Quinquennial'


FIBONACCI = 'Fibonacci'
CAMARILLA = 'Camarilla'
DEMARK = 'OFF'


kind = input.string(title='Type', defval='Camarilla', options=[DEMARK, FIBONACCI, CAMARILLA], inline='Pi', group='Pivots')
pivot_time_frame = input.string(title='', defval=MONTHLY, options=[AUTO, HOURLY, DAILY, WEEKLY, MONTHLY, QUARTERLY, YEARLY, BIYEARLY, TRIYEARLY, QUINQUENNIAL], inline='Pi', group='Pivots')
look_back = input.int(title='', defval=1, minval=1, maxval=5000, inline='Pi', group='Pivots', tooltip='No Of previous Levels')
is_daily_based = input.bool(title='Use Daily-based Values', defval=true, tooltip='When this option is unchecked, Pivot Points will use intraday data while calculating on intraday charts. If Extended Hours are displayed on the chart, they will be taken into account during the pivot level calculation. If intraday OHLC values are different from daily-based values (normal for stocks), the pivot levels will also differ.')

show_labels = false  //input.bool(title="Show Labels", type=input.bool, defval=false, inline = "labels")
position_labels = input.string('Left', '', options=['Left', 'Right'], inline='labels')

swing = input.bool(defval=false, title='Swing', inline='Swing', group='Settings', tooltip='Turn On Swing Hi/Lo')  //
prd = input.int(defval=10, title='Swing', inline='Swing', group='Settings ')
ColorSelector(c_) =>
    c_ == 'aqua' ? color.aqua : c_ == 'black' ? color.black : c_ == 'blue' ? color.blue : c_ == 'fuchsia' ? color.fuchsia : c_ == 'gray' ? color.gray : c_ == 'green' ? color.green : c_ == 'lime' ? color.lime : c_ == 'maroon' ? color.maroon : c_ == 'navy' ? color.navy : c_ == 'olive' ? color.olive : c_ == 'orange' ? color.orange : c_ == 'purple' ? color.purple : c_ == 'red' ? color.red : c_ == 'silver' ? color.silver : c_ == 'teal' ? color.teal : c_ == 'white' ? color.white : c_ == 'yellow' ? color.yellow : color.black

Hi_color = input.string(title='HH', defval='orange', options=['aqua', 'black', 'blue', 'fuchsia', 'gray', 'green', 'lime', 'maroon', 'navy', 'olive', 'orange', 'purple', 'red', 'silver', 'teal', 'white', 'yellow'],inline='Swing',group='Settings ')
Lo_color = input.string(title='LL', defval='black', options=['aqua', 'black', 'blue', 'fuchsia', 'gray', 'green', 'lime', 'maroon', 'navy', 'olive', 'orange', 'purple', 'red', 'silver', 'teal', 'white', 'yellow'],inline='Swing',group='Settings ')
showlast = input.bool(title='Show Only Last Period', defval=true, group='Settings ', tooltip='Shows only Today levels bar by bar')
showlabels = input.bool(title='Show Labels', defval=true, group='Settings ', tooltip='Shows labels on plotted pivots')
ltcol    = input.color(color.black,"Target Color",inline='labels')
lbcol    = input.color(color.yellow,"Breakout/down Color",inline='labels')
lstyle2 = input.string(title='CPR Style', options=['Solid', 'Circles', 'Cross'], defval='Solid', group='Style', tooltip='Change Line Style for CPR')
cmidon = input.bool(defval=false, title='Cam Mid   ', group='Settings ', tooltip='Turn Camarilla Mid')
cprturnon = input.bool(title='Turn On CPR', defval=false, group='Settings ', tooltip='Turn CPR on')
JP = input.bool(title='Just Pivot', defval=false, group='Settings ', tooltip='Show Only Pivot of the day')
PDHL = input.bool(false, 'Prev HiLo   ', inline='Settings ', group='Settings ')
PColor = input.color(color.orange, '', inline='Settings ', group='Settings ')
Pres = input.timeframe(defval='D', inline='Settings ', group='Settings ')
lvl = input.int(1, title='', inline='Settings ', group='Settings ')
vwaplot = input.bool(false, title='VWAP', inline='vwap', group='Settings ', tooltip='Turn on Vwap')
emaplot = input.bool(false, title='EMA on   ', inline='Settings ', group='Settings ', tooltip='Turns On 3 Ema\'s On Chart, Levels can be Edited')
choice = input.string(title='', defval='EMA', options=['EMA', 'SMA'], inline='Settings ', group='Settings ', tooltip='Select Either EMA or SMA from dropdown menu')
MAa = input.int(9, title='EMA', inline='EMA', minval=1, maxval=500, group='Settings ')
MAb = input.int(27, title=' ', inline='EMA', minval=1, maxval=500, group='Settings ')
MAc = input.int(111, title=' ', inline='EMA', minval=1, maxval=500, group='Settings ')
lw = input.int(1, title='Width', minval=1, maxval=3, inline='EMA', group='Settings ')
HammerInput = input.bool(true, 'Hammer', inline='ham', group='Price Action')
HangingManInput = input.bool(true, 'Hanging Man', group='Price Action')
InvertedHammerInput = input.bool(true, 'Inverted Hammer', inline='ham', group='Price Action')

vsr = input.bool(false, title='Show Volume Based S&R', group='Settings ', tooltip='Shows Volume Based Support and Resistance on Stocks and Futures')
ATRTsl = input.bool(false, 'Trailing SL', inline='atr', group='Settings ', tooltip='Trailing SL')

//Pivot Settings 
var DEF_COLOR = #FB8C00
var S3_COLOR = #000000
var S4_COLOR = #ff9800
var S5_COLOR = #ff0000
var arr_time = array.new_int()
var p = array.new_float()
p_show = input.bool(false, 'P‏  ‏  ‏  ‏  ‏  ‏  ‏  ‏', inline='P')
p_color = input.color(DEF_COLOR, '', inline='P')

var r1 = array.new_float()
var s1 = array.new_float()
s1r1_show = input.bool(false, 'S1/R1', inline='S1/R1')
s1r1_color = input.color(DEF_COLOR, '', inline='S1/R1')

var r2 = array.new_float()
var s2 = array.new_float()
s2r2_show = input.bool(true, 'S2/R2', inline='S2/R2')
s2r2_color = input.color(S3_COLOR, '', inline='S2/R2')

var r3 = array.new_float()
var s3 = array.new_float()
s3r3_show = input.bool(true, 'S3/R3', inline='S3/R3')
s3r3_color = input.color(S4_COLOR, '', inline='S3/R3')

var r4 = array.new_float()
var s4 = array.new_float()
s4r4_show = input.bool(true, 'S4/R4', inline='S4/R4')
s4r4_color = input.color(S5_COLOR, '', inline='S4/R4')

var r5 = array.new_float()
var s5 = array.new_float()
s5r5_show = input.bool(true, 'S5/R5', inline='S5/R5')
s5r5_color = input.color(S5_COLOR, '', inline='S5/R5')

pivotX_open = float(na)
pivotX_open := nz(pivotX_open[1], open)
pivotX_high = float(na)
pivotX_high := nz(pivotX_high[1], high)
pivotX_low = float(na)
pivotX_low := nz(pivotX_low[1], low)
pivotX_prev_open = float(na)
pivotX_prev_open := nz(pivotX_prev_open[1])
pivotX_prev_high = float(na)
pivotX_prev_high := nz(pivotX_prev_high[1])
pivotX_prev_low = float(na)
pivotX_prev_low := nz(pivotX_prev_low[1])
pivotX_prev_close = float(na)
pivotX_prev_close := nz(pivotX_prev_close[1])

get_pivot_resolution() =>
    resolution = 'M'
    if pivot_time_frame == AUTO
        if timeframe.isintraday
            resolution := timeframe.multiplier <= 15 ? 'D' : 'W'
            resolution
        else if timeframe.isweekly or timeframe.ismonthly
            resolution := '12M'
            resolution
    else if pivot_time_frame == HOURLY
        resolution := '240'
        resolution
    else if pivot_time_frame == DAILY
        resolution := 'D'
        resolution
    else if pivot_time_frame == WEEKLY
        resolution := 'W'
        resolution
    else if pivot_time_frame == MONTHLY
        resolution := 'M'
        resolution
    else if pivot_time_frame == QUARTERLY
        resolution := '3M'
        resolution
    else if pivot_time_frame == YEARLY or pivot_time_frame == BIYEARLY or pivot_time_frame == TRIYEARLY or pivot_time_frame == QUINQUENNIAL
        resolution := '12M'
        resolution
    resolution

var lines = array.new_line()
var labels = array.new_label()

draw_line(i, pivot, col) =>
    if array.size(arr_time) > 1
        array.push(lines, line.new(array.get(arr_time, i), array.get(pivot, i), array.get(arr_time, i + 1), array.get(pivot, i), color=col, xloc=xloc.bar_time))

draw_label(i, y, txt, txt_color) =>
    if show_labels
        offset = '‏  ‏  ‏  ‏  ‏'
        labels_align_str_left = position_labels == 'Left' ? txt + offset : offset + txt
        x = position_labels == 'Left' ? array.get(arr_time, i) : array.get(arr_time, i + 1)
        array.push(labels, label.new(x=x, y=y, text=labels_align_str_left, textcolor=txt_color, style=label.style_label_center, color=#00000000, xloc=xloc.bar_time))


fibonacci() =>
    pivotX_Median = (pivotX_prev_high + pivotX_prev_low + pivotX_prev_close) / 3
    pivot_range = pivotX_prev_high - pivotX_prev_low
    array.push(p, pivotX_Median)
    array.push(r1, pivotX_Median + 0.382 * pivot_range)
    array.push(s1, pivotX_Median - 0.382 * pivot_range)
    array.push(r2, pivotX_Median + 0.618 * pivot_range)
    array.push(s2, pivotX_Median - 0.618 * pivot_range)
    array.push(r3, pivotX_Median + 1 * pivot_range)
    array.push(s3, pivotX_Median - 1 * pivot_range)
    array.push(r4, pivotX_Median + 1.272 * pivot_range)
    array.push(s4, pivotX_Median - 1.272 * pivot_range)
    array.push(r5, pivotX_Median + 1.618 * pivot_range)
    array.push(s5, pivotX_Median - 1.618 * pivot_range)


camarilla() =>
    pivotX_Median = (pivotX_prev_high + pivotX_prev_low + pivotX_prev_close) / 3
    pivot_range = pivotX_prev_high - pivotX_prev_low
    H4 = pivotX_prev_close + pivot_range * 1.1 / 2
    H3 = pivotX_prev_close + pivot_range * 1.1 / 4
    H2 = pivotX_prev_close + pivot_range * 1.1 / 6
    H1 = pivotX_prev_close + pivot_range * 1.1 / 12
    L1 = pivotX_prev_close - pivot_range * 1.1 / 12
    L2 = pivotX_prev_close - pivot_range * 1.1 / 6
    L3 = pivotX_prev_close - pivot_range * 1.1 / 4
    L4 = pivotX_prev_close - pivot_range * 1.1 / 2
    L5 = L4 - 1.168 * (L3 - L4)
    H5 = H4 + 1.168 * (H4 - H3)
    H6 = pivotX_prev_high / pivotX_prev_low * pivotX_prev_close
    L6 = pivotX_prev_close - (H6 - pivotX_prev_close)
    array.push(p, pivotX_Median)
    array.push(r1, H2)
    array.push(s1, L2)
    array.push(r2, H3)
    array.push(s2, L3)
    array.push(r3, H4)
    array.push(s3, L4)
    array.push(r4, H5)
    array.push(s4, L5)
    array.push(r5, H6)
    array.push(s5, L6)

resolution = get_pivot_resolution()

[sec_open, sec_high, sec_low, sec_close] = request.security(syminfo.tickerid, resolution, [open, high, low, close], lookahead=barmerge.lookahead_on)
sec_open_gaps_on = request.security(syminfo.tickerid, resolution, open, gaps=barmerge.gaps_on, lookahead=barmerge.lookahead_on)

var number_of_years = 0
is_change_years = false
var custom_years_resolution = pivot_time_frame == BIYEARLY or pivot_time_frame == TRIYEARLY or pivot_time_frame == QUINQUENNIAL
if custom_years_resolution and ta.change(time(resolution))
    number_of_years += 1
    if pivot_time_frame == BIYEARLY and number_of_years % 2 == 0
        is_change_years := true
        number_of_years := 0
        number_of_years
    else if pivot_time_frame == TRIYEARLY and number_of_years % 3 == 0
        is_change_years := true
        number_of_years := 0
        number_of_years
    else if pivot_time_frame == QUINQUENNIAL and number_of_years % 5 == 0
        is_change_years := true
        number_of_years := 0
        number_of_years

var is_change = false
var uses_current_bar = timeframe.isintraday
var change_time = int(na)
is_time_change = ta.change(time(resolution)) and not custom_years_resolution or is_change_years
if is_time_change
    change_time := time
    change_time


if not uses_current_bar and is_time_change or uses_current_bar and not na(sec_open_gaps_on)
    if is_daily_based
        pivotX_prev_open := sec_open[1]
        pivotX_prev_high := sec_high[1]
        pivotX_prev_low := sec_low[1]
        pivotX_prev_close := sec_close[1]
        pivotX_open := sec_open
        pivotX_high := sec_high
        pivotX_low := sec_low
        pivotX_low
    else
        pivotX_prev_high := pivotX_high
        pivotX_prev_low := pivotX_low
        pivotX_prev_open := pivotX_open
        pivotX_open := open
        pivotX_high := high
        pivotX_low := low
        pivotX_prev_close := close[1]
        pivotX_prev_close

    if barstate.islast and not is_change and array.size(arr_time) > 0
        array.set(arr_time, array.size(arr_time) - 1, change_time)
    else
        array.push(arr_time, change_time)

    if kind == FIBONACCI
        fibonacci()
    else if kind == CAMARILLA
        camarilla()

    if array.size(arr_time) > look_back
        if array.size(arr_time) > 0
            array.shift(arr_time)
        if array.size(p) > 0 and p_show
            array.shift(p)
        if array.size(r1) > 0 and s1r1_show
            array.shift(r1)
        if array.size(s1) > 0 and s1r1_show
            array.shift(s1)
        if array.size(r2) > 0 and s2r2_show
            array.shift(r2)
        if array.size(s2) > 0 and s2r2_show
            array.shift(s2)
        if array.size(r3) > 0 and s3r3_show
            array.shift(r3)
        if array.size(s3) > 0 and s3r3_show
            array.shift(s3)
        if array.size(r4) > 0 and s4r4_show
            array.shift(r4)
        if array.size(s4) > 0 and s4r4_show
            array.shift(s4)
        if array.size(r5) > 0 and s5r5_show
            array.shift(r5)
        if array.size(s5) > 0 and s5r5_show
            array.shift(s5)
    is_change := true
    is_change
else
    if is_daily_based
        pivotX_high := math.max(pivotX_high, sec_high)
        pivotX_low := math.min(pivotX_low, sec_low)
        pivotX_low
    else
        pivotX_high := math.max(pivotX_high, high)
        pivotX_low := math.min(pivotX_low, low)
        pivotX_low

if barstate.islast and array.size(arr_time) > 0 and is_change
    is_change := false
    if array.size(arr_time) > 2 and custom_years_resolution
        last_pivot_time = array.get(arr_time, array.size(arr_time) - 1)
        prev_pivot_time = array.get(arr_time, array.size(arr_time) - 2)
        estimate_pivot_time = last_pivot_time - prev_pivot_time
        array.push(arr_time, last_pivot_time + estimate_pivot_time)
    else
        array.push(arr_time, time_close(resolution))

    for i = 0 to array.size(lines) - 1 by 1
        if array.size(lines) > 0
            line.delete(array.shift(lines))
        if array.size(lines) > 0
            label.delete(array.shift(labels))

    for i = 0 to array.size(arr_time) - 2 by 1
        if array.size(p) > 0 and p_show
            draw_line(i, p, p_color)
            draw_label(i, array.get(p, i), 'P', p_color)
        if array.size(r1) > 0 and s1r1_show
            draw_line(i, r1, s1r1_color)
            draw_label(i, array.get(r1, i), 'R1', s1r1_color)
        if array.size(s1) > 0 and s1r1_show
            draw_line(i, s1, s1r1_color)
            draw_label(i, array.get(s1, i), 'S1', s1r1_color)
        if array.size(r2) > 0 and s2r2_show
            draw_line(i, r2, s2r2_color)
            draw_label(i, array.get(r2, i), 'R2', s2r2_color)
        if array.size(s2) > 0 and s2r2_show
            draw_line(i, s2, s2r2_color)
            draw_label(i, array.get(s2, i), 'S2', s2r2_color)
        if array.size(r3) > 0 and s3r3_show
            draw_line(i, r3, s3r3_color)
            draw_label(i, array.get(r3, i), 'R3', s3r3_color)
        if array.size(s3) > 0 and s3r3_show
            draw_line(i, s3, s3r3_color)
            draw_label(i, array.get(s3, i), 'S3', s3r3_color)
        if array.size(r4) > 0 and s4r4_show
            draw_line(i, r4, s4r4_color)
            draw_label(i, array.get(r4, i), 'R4', s4r4_color)
        if array.size(s4) > 0 and s4r4_show
            draw_line(i, s4, s4r4_color)
            draw_label(i, array.get(s4, i), 'S4', s4r4_color)
        if array.size(r5) > 0 and s5r5_show
            draw_line(i, r5, s5r5_color)
            draw_label(i, array.get(r5, i), 'R5', s5r5_color)
        if array.size(s5) > 0 and s5r5_show
            draw_line(i, s5, s5r5_color)
            draw_label(i, array.get(s5, i), 'S5', s5r5_color)



////////////////////////////////
//float ph = na, float pl = na
//ph := pivothigh(prd, prd)
//pl := pivotlow(prd, prd)

//plotshape(ph and swing, text="H",  style=shape.labeldown, color=na, textcolor=color.red, location=location.abovebar , offset = -prd)
//plotshape(pl and swing, text="L",  style=shape.labeldown, color=na, textcolor=color.green, location=location.belowbar , offset = -prd)

//lft = input.int(30, 'Swing Hi', group='Settings '')
//rght = input.int(30, 'Swing Lo', group='Settings '')

hih = ta.pivothigh(high, prd, prd)
lol = ta.pivotlow(low, prd, prd)

top = ta.valuewhen(hih, high[prd], 0)
bot = ta.valuewhen(lol, low[prd], 0)

plot(swing ? top : na, color=top != top[1] ? na : ColorSelector(Hi_color), offset=-prd, editable=false)
plot(swing ? bot : na, color=bot != bot[1] ? na : ColorSelector(Lo_color), offset=-prd, editable=false)
////////ORB {
sess = input.session('0915-0945', title='ORB Period', inline='orb', group='Settings ')

t = time(timeframe.period, sess + ':1234567')
hide = timeframe.isintraday and timeframe.multiplier <= 10


is_newbar(res) =>
    ta.change(time(res)) != 0
in_session = not na(t)
is_first = in_session and not in_session[1]

orb_high = float(na)
orb_low = float(na)

if is_first
    orb_high := high
    orb_low := low
    orb_low
else
    orb_high := orb_high[1]
    orb_low := orb_low[1]
    orb_low
if high > orb_high and in_session
    orb_high := high
    orb_high
if low < orb_low and in_session
    orb_low := low
    orb_low

show15highlow = input.bool(title='ORB ', defval=false, inline='orb', group='Settings ')

plot(show15highlow ? orb_high : na, style=plot.style_circles, color=orb_high[1] != orb_high ? na : color.purple, title='IB High', linewidth=1, show_last=75)
plot(show15highlow ? orb_low : na, style=plot.style_circles, color=orb_low[1] != orb_low ? na : color.purple, title='IB Low', linewidth=1, show_last=75)

//}


hhtf = request.security(syminfo.tickerid, resolution, high[1], lookahead=barmerge.lookahead_on)
lhtf = request.security(syminfo.tickerid, resolution, low[1], lookahead=barmerge.lookahead_on)
chtf = request.security(syminfo.tickerid, resolution, close[1], lookahead=barmerge.lookahead_on)

rng = hhtf - lhtf

// is this last bar for HTF?
islast = showlast ? request.security(syminfo.tickerid, resolution, barstate.islast, lookahead=barmerge.lookahead_on) : true

// Line Style

linestyleL = plot.style_circles
///////Calculation Camarilla
H4 = chtf + rng * 1.1 / 2
H3 = chtf + rng * 1.1 / 4
H2 = chtf + rng * 1.1 / 6
H1 = chtf + rng * 1.1 / 12
L1 = chtf - rng * 1.1 / 12
L2 = chtf - rng * 1.1 / 6
L3 = chtf - rng * 1.1 / 4
L4 = chtf - rng * 1.1 / 2
L5 = L4 - 1.168 * (L3 - L4)  //L5 = chtf - (H5 - chtf)
H5 = H4 + 1.168 * (H4 - H3)  //H5 = (hhtf / lhtf) * chtf
H6 = hhtf / lhtf * chtf  //H6 = H5 + 1.168 * (H5 - H4) 
L6 = chtf - (H6 - chtf)  //L6 = chtf - (H6 - chtf)
SLbull = (H4 + H3) / 2
SLbear = (L4 + L3) / 2
mid = (H3 + L3) / 2
////////Color Settings

plot(islast and kind == CAMARILLA and cmidon ? mid : na, 'Mid-H3-L3', color=color.new(#000000, 60), linewidth=1, style=linestyleL, editable=false)
plot(islast and kind == CAMARILLA and cmidon ? SLbull : na, 'SLBull H3-H4', color=color.new(#ff0000, 60), linewidth=1, style=linestyleL, editable=false)
plot(islast and kind == CAMARILLA and cmidon ? SLbear : na, 'SLBear L3-L4', color=color.new(#388e3c, 60), linewidth=1, style=linestyleL, editable=false)

// Label for S/R
mndr = time - time[1]
mndr := ta.change(mndr) > 0 ? mndr[1] : mndr

Round_it(valu) =>
    a = 0
    num = syminfo.mintick
    s = valu
    if na(s)
        s := syminfo.mintick
        s
    if num < 1
        for i = 1 to 20 by 1
            num *= 10
            if num > 1
                break
            a += 1
            a

    for x = 1 to a by 1
        s *= 10
        s
    s := math.round(s)
    for x = 1 to a by 1
        s /= 10
        s
    s := s < syminfo.mintick ? syminfo.mintick : s
    s

// Labels
if showlabels and kind == CAMARILLA
    var label s3label = na
    var label s4label = na
    var label s5label = na
    var label s6label = na
    var label r3label = na
    var label r4label = na
    var label r5label = na
    var label r6label = na

    label.delete(s3label)
    label.delete(s4label)
    label.delete(s5label)
    label.delete(s6label)
    label.delete(r3label)
    label.delete(r4label)
    label.delete(r5label)
    label.delete(r6label)
    s3label := label.new(x=time + mndr * 20, y=L3, text='Buy Reversal •' + str.tostring(Round_it(L3)), color=color.new(#000000, 100), textcolor=color.green, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    s4label := label.new(x=time + mndr * 20, y=L4, text='Break Down •' + str.tostring(Round_it(L4)), color=color.new(#000000, 100), textcolor=lbcol, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    s5label := label.new(x=time + mndr * 20, y=L5, text='Target •' + str.tostring(Round_it(L5)), color=color.new(#000000, 100), textcolor=ltcol, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    s6label := label.new(x=time + mndr * 20, y=L6, text='Target •' + str.tostring(Round_it(L6)), color=color.new(#000000, 100), textcolor=ltcol, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    r3label := label.new(x=time + mndr * 20, y=H3, text='Sell reversal •' + str.tostring(Round_it(H3)), color=color.new(#000000, 100), textcolor=color.red, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    r4label := label.new(x=time + mndr * 20, y=H4, text='Breakout •' + str.tostring(Round_it(H4)), color=color.new(#000000, 100), textcolor=lbcol, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    r5label := label.new(x=time + mndr * 20, y=H5, text='Target •' + str.tostring(Round_it(H5)), color=color.new(#000000, 100), textcolor=ltcol, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    r6label := label.new(x=time + mndr * 20, y=H6, text='Target •' + str.tostring(Round_it(H6)), color=color.new(#000000, 100), textcolor=ltcol, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    r6label

//////Central Pivot
Pivot = (hhtf + lhtf + chtf) / 3
BC = (hhtf + lhtf) / 2
TC = Pivot - BC + Pivot
//LineStyle CPR
linestylee = lstyle2 == 'Solid' ? plot.style_line : lstyle2 == 'Circles' ? plot.style_circles : lstyle2 == 'Cross' ? plot.style_cross : na


plot(islast and cprturnon ? TC : na, title='TC', color=color.new(color.blue, 0), linewidth=1, style=linestylee)
plot(islast and cprturnon ? Pivot : na, title='Pivot', color=color.new(color.red, 0), linewidth=1, style=linestylee)
plot(islast and cprturnon ? BC : na, title='BC', color=color.new(color.blue, 0), linewidth=1, style=linestylee)
plot(islast and JP ? Pivot : na, title='JPivot', color=color.new(color.red, 0), linewidth=2, style=linestylee)
if showlabels and JP
    var label jplabel = na

    label.delete(jplabel)
    jplabel := label.new(x=time + mndr * 20, y=Pivot, text='Pivot  •' + str.tostring(Round_it(Pivot)), color=color.new(#000000, 100), textcolor=color.black, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    chtf
///////Day High Low//////
phhtf = request.security(syminfo.tickerid, Pres, high[lvl], lookahead=barmerge.lookahead_on)
plhtf = request.security(syminfo.tickerid, Pres, low[lvl], lookahead=barmerge.lookahead_on)
pchtf = request.security(syminfo.tickerid, Pres, close[lvl], lookahead=barmerge.lookahead_on)
islast2 = showlast ? request.security(syminfo.tickerid, Pres, barstate.islast, lookahead=barmerge.lookahead_on) : true
plot(islast2 and PDHL ? phhtf : na, title='Day High', color=PColor, linewidth=2, style=linestylee)
plot(islast2 and PDHL ? plhtf : na, title='Day Low', color=PColor, linewidth=2, style=linestylee)
plot(islast2 and PDHL ? pchtf : na, title='Day Close', color=PColor, linewidth=2, style=linestylee)
if showlabels and PDHL
    var label pdhlabel = na
    var label pdllabel = na
    var label pdclabel = na

    label.delete(pdhlabel)
    label.delete(pdllabel)
    label.delete(pdclabel)
    pdhlabel := label.new(x=time + mndr * 20, y=phhtf, text='PDH  •' + str.tostring(Round_it(phhtf)), color=color.new(#000000, 100), textcolor=PColor, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    pdllabel := label.new(x=time + mndr * 20, y=plhtf, text='PDL  •' + str.tostring(Round_it(plhtf)), color=color.new(#000000, 100), textcolor=PColor, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    pdclabel := label.new(x=time + mndr * 20, y=pchtf, text='PDC  •' + str.tostring(Round_it(pchtf)), color=color.new(#000000, 100), textcolor=PColor, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    pdclabel
//////Fibo Pivot

pivot = (hhtf + lhtf + chtf) / 3.0
R1 = pivot + 0.382 * rng
S1 = pivot - 0.382 * rng
R2 = pivot + 0.618 * rng
S2 = pivot - 0.618 * rng
R3 = pivot + rng
S3 = pivot - rng
R4 = pivot + 1.272 * rng
S4 = pivot - 1.272 * rng
R5 = pivot + 1.618 * rng
S5 = pivot - 1.618 * rng
R6 = pivot + 2.058 * rng
S6 = pivot - 2.058 * rng
R7 = pivot + 2.618 * rng
S7 = pivot - 2.618 * rng

if showlabels and kind == FIBONACCI
    var label fs1label = na
    var label fs2label = na
    var label fs3label = na
    var label fs4label = na
    var label plabel = na
    var label fr1label = na
    var label fr2label = na
    var label fr3label = na
    var label fr4label = na

    label.delete(fs1label)
    label.delete(fs2label)
    label.delete(fs3label)
    label.delete(fs4label)
    label.delete(plabel)
    label.delete(fr1label)
    label.delete(fr2label)
    label.delete(fr3label)
    label.delete(fr4label)
    fs1label := label.new(x=time + mndr * 20, y=S1, text='0.382  ' + str.tostring(Round_it(S1)), color=color.new(#000000, 100), textcolor=color.green, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    fs2label := label.new(x=time + mndr * 20, y=S2, text='0.618 ' + str.tostring(Round_it(S2)), color=color.new(#000000, 100), textcolor=color.black, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    fs3label := label.new(x=time + mndr * 20, y=S3, text='100 ' + str.tostring(Round_it(S3)), color=color.new(#000000, 100), textcolor=color.black, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    fs4label := label.new(x=time + mndr * 20, y=S4, text='1.272 ' + str.tostring(Round_it(S4)), color=color.new(#000000, 100), textcolor=color.black, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    plabel := label.new(x=time + mndr * 20, y=pivot, text='Pivot ' + str.tostring(Round_it(pivot)), color=color.new(#000000, 100), textcolor=color.silver, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    fr1label := label.new(x=time + mndr * 20, y=R1, text='0.382 ' + str.tostring(Round_it(R1)), color=color.new(#000000, 100), textcolor=color.red, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    fr2label := label.new(x=time + mndr * 20, y=R2, text='0.618 ' + str.tostring(Round_it(R2)), color=color.new(#000000, 100), textcolor=color.black, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    fr3label := label.new(x=time + mndr * 20, y=R3, text='100 ' + str.tostring(Round_it(R3)), color=color.new(#000000, 100), textcolor=color.black, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    fr4label := label.new(x=time + mndr * 20, y=R4, text='1.272 ' + str.tostring(Round_it(R4)), color=color.new(#000000, 100), textcolor=color.black, style=label.style_label_left, xloc=xloc.bar_time, yloc=yloc.price)
    fr4label

PAB = input.bool(true, 'Price Action Bars', group='Settings ')
//Dark Cloud
DRKC = open[1] < close[1] ? open > high[1] ? close < close[1] - (close[1] - open[1]) / 2 ? close > open[1] ? #dbff01 : na : na : na : na
barcolor(PAB ? DRKC : na, title='Dark Cloud')

//Bearish Engulfing
BrEng = close < open[1] ? low < low[1] ? high > high[1] ? open >= open[1] ? #ff0000 : na : na : na : na
barcolor(PAB ? BrEng : na, title='Bearish Engulf')


//Bullish Engulfing
BuEng = low < low[1] ? high > high[1] ? open <= open[1] ? close > open[1] ? #00ff0a : na : na : na : na
barcolor(PAB ? BuEng : na, title='Bullish Engulf')

BearishENG = open[1] < close[1] ? close < open[1] ? open > close[1] ? #ff0000 : na : na : na
barcolor(PAB ? BearishENG : na, title='Bearish 2 Engulf')
BullishENG = open[1] > close[1] ? close > open[1] ? open < close[1] ? #00ff0a : na : na : na
barcolor(PAB ? BullishENG : na, title='Bullish 2 Engulf')

//Three White Soldiers
//TWS = close > open ? close[1] > open[1] ? close[2] > open[2] ? close > high[1] ? close[1] > high[2] ? open < close[1] ? open[1] < close[2] ? (high - close ) * 3 < close - open ? (high[1] - close[1]) * 3 < close[1] - open[1] ? (high[2] - close[2]) * 3 < close[2] - open[2] ? #66ff00 : na : na : na : na : na : na:na : na : na : na
//barcolor(PAB ? TWS : na, title="Three white soliders")
//TBC = close < open ? close[1] < open[1] ? close[2] < open[2] ? close < low[1] ? close[1] < low[2] ? open > close[1] ? open[1] > close[2] ? (close - low) * 3 < open - close ? (close[1] - low[1]) * 3 < open[1] - close[1] ? (close[2] - low[2]) * 3 < open[2] - close[2] ? #ff6600 : na : na : na : na : na : na:na : na : na : na
//barcolor (PAB ? TBC : na, title = "Three Black Crows")
/////VWAP////MVWAP
Length = input.int(50, title='MVWAP', inline='vwap', group='Settings ')
mvwap = ta.ema(ta.vwap, Length)
plot(vwaplot ? mvwap : na, linewidth=2, title='MVWAP', style=plot.style_line, color=color.new(color.purple, 0))

plot(ta.vwap and vwaplot ? ta.vwap : na, linewidth=lw, title='VWAP', color=color.new(#FF7000, 0))

//////EMA
plot(emaplot and choice == 'EMA' ? ta.ema(close, MAa) : emaplot and choice == 'SMA' ? ta.sma(close, MAa) : na, title='Fast MA', color=color.new(color.green, 0), linewidth=lw)
plot(emaplot and choice == 'EMA' ? ta.ema(close, MAb) : emaplot and choice == 'SMA' ? ta.sma(close, MAb) : na, title='Mid MA', color=color.new(color.black, 0), linewidth=lw)
plot(emaplot and choice == 'EMA' ? ta.ema(close, MAc) : emaplot and choice == 'SMA' ? ta.sma(close, MAc) : na, title='Slow MA', color=color.new(color.red, 0), linewidth=lw)


////////////Volume Based Support Resistance
Vlength = input.int(20, minval=1, group='Volume S/R Settings')
Vchange = volume / volume[1] - 1
stdev = ta.stdev(Vchange, Vlength)
difference = Vchange / stdev[1]
Treshold = input(5)
zero = 0
signal = math.abs(difference)
vstylee = plot.style_circles

leveluphi = ta.valuewhen(signal > Treshold, high[1], 0)
leveluplo = ta.valuewhen(signal > Treshold, low[1], 0)

//plot(UpperTreshold, color=black)
pv1 = plot(vsr and leveluphi ? leveluphi : na, title='LevelHi', style=vstylee, color=color.new(color.blue, 0))
pv2 = plot(vsr and leveluplo ? leveluplo : na, title='Levello', style=vstylee, color=color.new(color.blue, 0))
fill(pv1, pv2, color=color.new(color.black, 50), title='Fill')
////////////////////////
Ecandle = input.bool(false, 'Indecisive-Candle', group='Settings ', tooltip='Shows 50% Candles')

cand = high - low
bodyr = open - close

candle = bodyr * 100 / cand

barcolor(Ecandle and candle > -50 and candle < 50 ? #0b00ff : na)
///// 

///////// Day Range
On = input.bool(false, 'Day Range', group='Settings ')

fill(plot1=plot(On and islast ? H4 : na, color=color.new(#ff7700, 80), editable=false), plot2=plot(On and islast ? R2 : na, color=color.new(#ff7700, 100), editable=false), color=color.new(#ff7700, 75))
fill(plot1=plot(On and islast ? L4 : na, color=color.new(#000000, 80), editable=false), plot2=plot(On and islast ? S2 : na, color=color.new(#000000, 100), editable=false), color=color.new(#000000, 75))

//Price Action {
TR1 = input.int(27, title='SMA to determine Trend', minval=1, maxval=500)
TR2 = input.int(111, title='SMA to determine Trend', minval=1, maxval=500)
CS = input.int(14, title='Candle Strength', minval=14, maxval=50)
confirm = barstate.isconfirmed
C_DownTrend = true
C_UpTrend = true
var trendRule1 = 'EMA50'
var trendRule2 = 'EMA50, EMA200'
var trendRule = input.string(trendRule1, '', options=[trendRule1, trendRule2, 'No detection'], inline='RT', group='Price Action')

if trendRule == trendRule1
    priceAvg = ta.ema(close, TR1)
    C_DownTrend := close < priceAvg
    C_UpTrend := close > priceAvg
    C_UpTrend

if trendRule == trendRule2
    sma200 = ta.ema(close, TR2)
    sma50 = ta.ema(close, TR1)
    C_DownTrend := close < sma50 and sma50 < sma200
    C_UpTrend := close > sma50 and sma50 > sma200
    C_UpTrend
C_Len = CS  // ema depth for bodyAvg
C_ShadowPercent = 5.0  // size of shadows
C_ShadowEqualsPercent = 100.0
C_DojiBodyPercent = 5.0
C_Factor = 2.0  // shows the number of times the shadow dominates the candlestick body

C_BodyHi = math.max(close, open)
C_BodyLo = math.min(close, open)
C_Body = C_BodyHi - C_BodyLo
C_BodyAvg = ta.ema(C_Body, C_Len)
C_SmallBody = C_Body < C_BodyAvg
C_LongBody = C_Body > C_BodyAvg
C_UpShadow = high - C_BodyHi
C_DnShadow = C_BodyLo - low
C_HasUpShadow = C_UpShadow > C_ShadowPercent / 100 * C_Body
C_HasDnShadow = C_DnShadow > C_ShadowPercent / 100 * C_Body
C_WhiteBody = open < close
C_BlackBody = open > close
C_Range = high - low
C_IsInsideBar = C_BodyHi[1] > C_BodyHi and C_BodyLo[1] < C_BodyLo
C_BodyMiddle = C_Body / 2 + C_BodyLo
C_ShadowEquals = C_UpShadow == C_DnShadow or math.abs(C_UpShadow - C_DnShadow) / C_DnShadow * 100 < C_ShadowEqualsPercent and math.abs(C_DnShadow - C_UpShadow) / C_UpShadow * 100 < C_ShadowEqualsPercent
C_IsDojiBody = C_Range > 0 and C_Body <= C_Range * C_DojiBodyPercent / 100
C_Doji = C_IsDojiBody and C_ShadowEquals

patternLabelPosLow = low - ta.atr(30) * 0.99
patternLabelPosHigh = high + ta.atr(30) * 0.99


text_color_bullish = input(color.green, 'Text Color Bull')
text_color_bearish = input(color.red, 'Text color Bear')
CandleType = input.string(title='Pattern Type', defval='Both', options=['Bullish', 'Bearish', 'Both'])


C_HammerBullishNumberOfCandles = 1
C_HammerBullish = false
if C_SmallBody and C_Body > 0 and C_BodyLo > hl2 and C_DnShadow >= C_Factor * C_Body and not C_HasUpShadow and confirm
    if C_DownTrend
        C_HammerBullish := true
        C_HammerBullish
alertcondition(C_HammerBullish, title='Hammer – Bullish', message='New Hammer – Bullish pattern detected')
if C_HammerBullish and HammerInput and ('Bullish' == CandleType or CandleType == 'Both')

    var ttBullishHammer = 'Hammer\nHammer candlesticks form when a security moves lower after the open, but continues to rally into close above the intraday low. The candlestick that you are left with will look like a square attached to a long stick-like figure. This candlestick is called a Hammer if it happens to form during a decline.'
    label.new(bar_index, patternLabelPosLow, text='H', style=label.style_none, textcolor=text_color_bullish, tooltip=ttBullishHammer)


C_HangingManBearishNumberOfCandles = 1
C_HangingManBearish = false
if C_SmallBody and C_Body > 0 and C_BodyLo > hl2 and C_DnShadow >= C_Factor * C_Body and not C_HasUpShadow and confirm
    if C_UpTrend
        C_HangingManBearish := true
        C_HangingManBearish
alertcondition(C_HangingManBearish, title='Hanging Man – Bearish', message='New Hanging Man – Bearish pattern detected')
if C_HangingManBearish and HangingManInput and ('Bearish' == CandleType or CandleType == 'Both')

    var ttBearishHangingMan = 'Hanging Man\nWhen a specified security notably moves lower after the open, but continues to rally to close above the intraday low, a Hanging Man candlestick will form. The candlestick will resemble a square, attached to a long stick-like figure. It is referred to as a Hanging Man if the candlestick forms during an advance.'
    label.new(bar_index, patternLabelPosHigh, text='HM', style=label.style_none, textcolor=text_color_bearish, tooltip=ttBearishHangingMan)


C_InvertedHammerBullishNumberOfCandles = 1
C_InvertedHammerBullish = false
if C_SmallBody and C_Body > 0 and C_BodyHi < hl2 and C_UpShadow >= C_Factor * C_Body and not C_HasDnShadow and confirm
    if C_DownTrend
        C_InvertedHammerBullish := true
        C_InvertedHammerBullish
alertcondition(C_InvertedHammerBullish, title='Inverted Hammer – Bullish', message='New Inverted Hammer – Bullish pattern detected')
if C_InvertedHammerBullish and InvertedHammerInput and ('Bullish' == CandleType or CandleType == 'Both')

    var ttBullishInvertedHammer = 'Inverted Hammer\nIf in a downtrend, then the open is lower. When it eventually trades higher, but closes near its open, it will look like an inverted version of the Hammer Candlestick. This is a one-day bullish reversal pattern.'
    label.new(bar_index, patternLabelPosLow, text='IH', style=label.style_none, textcolor=text_color_bullish, tooltip=ttBullishInvertedHammer)



// ATR Trailing SL {
nATRPeriod = input.int(5, 'Period', inline='atr', group='Settings ')
nATRMultip = input.float(3.5, 'Multi', inline='atr', group='Settings ')
xATR = ta.atr(nATRPeriod)
nLoss = nATRMultip * xATR
xATRTrailingStop = 0.0
iff_1 = close > nz(xATRTrailingStop[1], 0) ? close - nLoss : close + nLoss
iff_2 = close < nz(xATRTrailingStop[1], 0) and close[1] < nz(xATRTrailingStop[1], 0) ? math.min(nz(xATRTrailingStop[1]), close + nLoss) : iff_1
xATRTrailingStop := close > nz(xATRTrailingStop[1], 0) and close[1] > nz(xATRTrailingStop[1], 0) ? math.max(nz(xATRTrailingStop[1]), close - nLoss) : iff_2
col = close < xATRTrailingStop[1] ? color.silver : color.black
plot(ATRTsl ? xATRTrailingStop[1] : na, color=col, title='ATR Trailing Stop')  //}

//Table {
var table info = table.new(position.top_center, 1, 1)
var table logo = table.new(position.bottom_right, 1, 1)
if barstate.islast
    table.cell(logo, 0, 0, 'Tali ', text_size=size.normal, text_color=color.orange)
    table.cell(info, 0, 0, 't.me/tali1991', text_size=size.small, text_color=color.black)
//}
//{RSI col
rsicol = input.bool(false, title='Show RSI colors?', group='Settings ', tooltip='Show RSI Levels On Bars')

srcRSI = close
lenRSI = input.int(14, minval=1, title='RSI Length', group='RSI Settings')
up = ta.rma(math.max(ta.change(srcRSI), 0), lenRSI)
down = ta.rma(-math.min(ta.change(srcRSI), 0), lenRSI)
rsi = down == 0 ? 100 : up == 0 ? 0 : 100 - 100 / (1 + up / down)
//coloring method below
srcRSI1 = close
lenRSI1 = input.int(60, minval=1, title='Over Bought', group='RSI Settings')
srcRSI2 = close
lenRSI2 = input.int(40, minval=1, title='Over Sold', group='RSI Settings')
isup() =>
    rsi > lenRSI1
isdown() =>
    rsi < lenRSI2
barcolor(rsicol and isup() ? color.green : rsicol and isdown() ? color.red : na)  // }
//Mid Point {
Mid = input.bool(true, 'Mid Point', group='Settings ')
plotchar(Mid ? hl2 : na, char='•', color=color.new(color.red, 0), location=location.absolute, size=size.tiny, offset=1, show_last=5)  //} 
//BollingerBands {
bbon = input.bool(false, title='BollingerBands', inline='BB', group='Settings ')
bblength = input.int(27, '', minval=1, inline='BB', group='Settings ')
bbsrc = input.source(close, title='', inline='BB', group='Settings ')
mult = input.float(2.0, minval=0.001, maxval=50, title='', inline='BB', group='Settings ')
basis = ta.sma(bbsrc, bblength)
dev = mult * ta.stdev(bbsrc, bblength)
upper = basis + dev
lower = basis - dev
offset = input.int(0, 'BB Offset', minval=-500, maxval=500)
plot(bbon ? basis : na, 'Basis', color=color.new(#872323, 0), offset=offset)
b1 = plot(bbon ? upper : na, 'Upper', color=color.new(color.teal, 0), offset=offset)
b2 = plot(bbon ? lower : na, 'Lower', color=color.new(color.teal, 0), offset=offset)
fill(b1, b2, title='Background', color=color.new(#198787, 95))  //}

////////////Trendlines Taken from Lonesomeblue
startyear = input(defval=2020, title='Start Year',group='Trendline')
startmonth = input(defval=1, title='Start Month',group='Trendline')
startday = input(defval=1, title='Start day',group='Trendline')
prdl = input.int(defval=10, title='Pivot Period', minval=10, maxval=50,group='Trendline')
PPnum = input.int(defval=3, title='Number of Pivot Points to check', minval=2, maxval=8,group='Trendline')
utcol = input.color(defval=color.lime, title='Colors', inline='tcol',group='Trendline')
dtcol = input.color(defval=color.red, title='', inline='tcol',group='Trendline')

float ph = ta.pivothigh(prdl, prdl)
float pl = ta.pivotlow(prdl, prdl)

var tval = array.new_float(PPnum)
var tpos = array.new_int(PPnum)
var bval = array.new_float(PPnum)
var bpos = array.new_int(PPnum)

add_to_array(apointer1, apointer2, val) =>
    array.unshift(apointer1, val)
    array.unshift(apointer2, bar_index)
    array.pop(apointer1)
    array.pop(apointer2)

if ph
    add_to_array(tval, tpos, ph)

if pl
    add_to_array(bval, bpos, pl)

// line definitions
maxline = 3
var bln = array.new_line(maxline, na)
var tln = array.new_line(maxline, na)

// loop for pivot points to check if there is possible trend line
countlinelo = 0
countlinehi = 0

starttime = timestamp(startyear, startmonth, startday, 0, 0, 0)

if time >= starttime
    for x = 0 to maxline - 1 by 1
        line.delete(array.get(bln, x))
        line.delete(array.get(tln, x))
    for p1 = 0 to PPnum - 2 by 1
        uv1 = 0.0
        uv2 = 0.0
        up1 = 0
        up2 = 0
        if countlinelo <= maxline
            for p2 = PPnum - 1 to p1 + 1 by 1
                val1 = array.get(bval, p1)
                val2 = array.get(bval, p2)
                pos1 = array.get(bpos, p1)
                pos2 = array.get(bpos, p2)
                if val1 > val2
                    diff = (val1 - val2) / (pos1 - pos2)
                    hline = val2 + diff
                    lloc = bar_index
                    lval = low
                    valid = true
                    for x = pos2 + 1 - prd to bar_index by 1
                        if close[bar_index - x] < hline
                            valid := false
                            break
                        lloc := x
                        lval := hline
                        hline += diff
                        hline


                    if valid
                        uv1 := hline - diff
                        uv2 := val2
                        up1 := lloc
                        up2 := pos2
                        break

        dv1 = 0.0
        dv2 = 0.0
        dp1 = 0
        dp2 = 0
        if countlinehi <= maxline
            for p2 = PPnum - 1 to p1 + 1 by 1
                val1 = array.get(tval, p1)
                val2 = array.get(tval, p2)
                pos1 = array.get(tpos, p1)
                pos2 = array.get(tpos, p2)
                if val1 < val2
                    diff = (val2 - val1) / float(pos1 - pos2)
                    hline = val2 - diff
                    lloc = bar_index
                    lval = high
                    valid = true
                    for x = pos2 + 1 - prd to bar_index by 1
                        if close[bar_index - x] > hline
                            valid := false
                            break
                        lloc := x
                        lval := hline
                        hline -= diff
                        hline

                    if valid
                        dv1 := hline + diff
                        dv2 := val2
                        dp1 := lloc
                        dp2 := pos2
                        break

        // if there is continues uptrend line then draw it
        if up1 != 0 and up2 != 0 and countlinelo < maxline
            countlinelo += 1
            array.set(bln, countlinelo - 1, line.new(up2 - prd, uv2, up1, uv1, color=utcol))

        // if there is continues downtrend line then draw it
        if dp1 != 0 and dp2 != 0 and countlinehi < maxline
            countlinehi += 1
            array.set(tln, countlinehi - 1, line.new(dp2 - prd, dv2, dp1, dv1, color=dtcol))

//divergences
pivotprd = input.int(defval=5, title='Pivot Period', minval=1, maxval=50)
source = input.string(defval='Close', title='Source for Pivot Points', options=['Close', 'High/Low'])
searchdiv = input.string(defval='Regular', title='Divergence Type', options=['Regular', 'Hidden', 'Regular/Hidden'])
showindis = input.string(defval='Full', title='Show Indicator Names', options=['Full', 'First Letter', 'Don\'t Show'])
showlimit = input.int(1, title='Minimum Number of Divergence', minval=1, maxval=11)
maxpp = input.int(defval=10, title='Maximum Pivot Points to Check', minval=1, maxval=20)
maxbars = input.int(defval=100, title='Maximum Bars to Check', minval=30, maxval=200)
shownum = input(defval=false, title='Show Divergence Number')
showlastd = input(defval=true, title='Show Only Last Divergence')
dontconfirm = input(defval=false, title='Don\'t Wait for Confirmation')
showlines = input(defval=true, title='Show Divergence Lines')
showpivot = input(defval=false, title='Show Pivot Points')
calcrsi = input(defval=true, title='RSI')
externalindi = input(defval=close, title='External Indicator')
pos_reg_div_col = input(defval=color.yellow, title='Positive Regular Divergence')
neg_reg_div_col = input(defval=color.navy, title='Negative Regular Divergence')
pos_hid_div_col = input(defval=color.lime, title='Positive Hidden Divergence')
neg_hid_div_col = input(defval=color.red, title='Negative Hidden Divergence')
pos_div_text_col = input(defval=color.black, title='Positive Divergence Text Color')
neg_div_text_col = input(defval=color.white, title='Negative Divergence Text Color')
reg_div_l_style_ = input.string(defval='Solid', title='Regular Divergence Line Style', options=['Solid', 'Dashed', 'Dotted'])
hid_div_l_style_ = input.string(defval='Dashed', title='Hdden Divergence Line Style', options=['Solid', 'Dashed', 'Dotted'])
reg_div_l_width = input.int(defval=2, title='Regular Divergence Line Width', minval=1, maxval=5)
hid_div_l_width = input.int(defval=1, title='Hidden Divergence Line Width', minval=1, maxval=5)


// set line styles
var reg_div_l_style = reg_div_l_style_ == 'Solid' ? line.style_solid : reg_div_l_style_ == 'Dashed' ? line.style_dashed : line.style_dotted
var hid_div_l_style = hid_div_l_style_ == 'Solid' ? line.style_solid : hid_div_l_style_ == 'Dashed' ? line.style_dashed : line.style_dotted


// get indicators
rsid = ta.rsi(close, 14)  // RSI

// keep indicators names and colors in arrays
var indicators_name = array.new_string(11)
var div_colors = array.new_color(4)
if barstate.isfirst
    array.set(indicators_name, 2, showindis == 'Full' ? 'RSI' : 'E')
    //colors
    array.set(div_colors, 0, pos_reg_div_col)
    array.set(div_colors, 1, neg_reg_div_col)
    array.set(div_colors, 2, pos_hid_div_col)
    array.set(div_colors, 3, neg_hid_div_col)

// Check if we get new Pivot High Or Pivot Low
float pih = ta.pivothigh(source == 'Close' ? close : high, pivotprd, pivotprd)
float pil = ta.pivotlow(source == 'Close' ? close : low, pivotprd, pivotprd)
plotshape(pih and showpivot, text='H', style=shape.labeldown, color=color.new(color.white, 100), textcolor=color.new(color.red, 0), location=location.abovebar, offset=-pivotprd)
plotshape(pil and showpivot, text='L', style=shape.labelup, color=color.new(color.white, 100), textcolor=color.new(color.lime, 0), location=location.belowbar, offset=-pivotprd)

// keep values and positions of Pivot Highs/Lows in the arrays
var int maxarraysize = 20
var ph_positions = array.new_int(maxarraysize, 0)
var pl_positions = array.new_int(maxarraysize, 0)
var ph_vals = array.new_float(maxarraysize, 0.)
var pl_vals = array.new_float(maxarraysize, 0.)

// add PHs to the array
if pih
    array.unshift(ph_positions, bar_index)
    array.unshift(ph_vals, pih)
    if array.size(ph_positions) > maxarraysize
        array.pop(ph_positions)
        array.pop(ph_vals)

// add PLs to the array
if pil
    array.unshift(pl_positions, bar_index)
    array.unshift(pl_vals, pil)
    if array.size(pl_positions) > maxarraysize
        array.pop(pl_positions)
        array.pop(pl_vals)

// functions to check Regular Divergences and Hidden Divergences

// function to check positive regular or negative hidden divergence
// cond == 1 => positive_regular, cond == 2=> negative_hidden
positive_regular_positive_hidden_divergence(src, cond) =>
    divlen = 0
    prsc = source == 'Close' ? close : low
    // if indicators higher than last value and close price is higher than las close 
    if dontconfirm or src > src[1] or close > close[1]
        startpoint = dontconfirm ? 0 : 1  // don't check last candle
        // we search last 15 PPs
        for x = 0 to maxpp - 1 by 1
            len = bar_index - array.get(pl_positions, x) + pivotprd
            // if we reach non valued array element or arrived 101. or previous bars then we don't search more
            if array.get(pl_positions, x) == 0 or len > maxbars
                break
            if len > 5 and (cond == 1 and src[startpoint] > src[len] and prsc[startpoint] < nz(array.get(pl_vals, x)) or cond == 2 and src[startpoint] < src[len] and prsc[startpoint] > nz(array.get(pl_vals, x)))
                slope1 = (src[startpoint] - src[len]) / (len - startpoint)
                virtual_line1 = src[startpoint] - slope1
                slope2 = (close[startpoint] - close[len]) / (len - startpoint)
                virtual_line2 = close[startpoint] - slope2
                arrived = true
                for y = 1 + startpoint to len - 1 by 1
                    if src[y] < virtual_line1 or nz(close[y]) < virtual_line2
                        arrived := false
                        break
                    virtual_line1 -= slope1
                    virtual_line2 -= slope2
                    virtual_line2

                if arrived
                    divlen := len
                    break
    divlen

// function to check negative regular or positive hidden divergence
// cond == 1 => negative_regular, cond == 2=> positive_hidden
negative_regular_negative_hidden_divergence(src, cond) =>
    divlen = 0
    prsc = source == 'Close' ? close : high
    // if indicators higher than last value and close price is higher than las close 
    if dontconfirm or src < src[1] or close < close[1]
        startpoint = dontconfirm ? 0 : 1  // don't check last candle
        // we search last 15 PPs
        for x = 0 to maxpp - 1 by 1
            len = bar_index - array.get(ph_positions, x) + pivotprd
            // if we reach non valued array element or arrived 101. or previous bars then we don't search more
            if array.get(ph_positions, x) == 0 or len > maxbars
                break
            if len > 5 and (cond == 1 and src[startpoint] < src[len] and prsc[startpoint] > nz(array.get(ph_vals, x)) or cond == 2 and src[startpoint] > src[len] and prsc[startpoint] < nz(array.get(ph_vals, x)))
                slope1 = (src[startpoint] - src[len]) / (len - startpoint)
                virtual_line1 = src[startpoint] - slope1
                slope2 = (close[startpoint] - nz(close[len])) / (len - startpoint)
                virtual_line2 = close[startpoint] - slope2
                arrived = true
                for y = 1 + startpoint to len - 1 by 1
                    if src[y] > virtual_line1 or nz(close[y]) > virtual_line2
                        arrived := false
                        break
                    virtual_line1 -= slope1
                    virtual_line2 -= slope2
                    virtual_line2

                if arrived
                    divlen := len
                    break
    divlen

// calculate 4 types of divergence if enabled in the options and return divergences in an array
calculate_divs(cond, indicator_1) =>
    divs = array.new_int(4, 0)
    array.set(divs, 0, cond and (searchdiv == 'Regular' or searchdiv == 'Regular/Hidden') ? positive_regular_positive_hidden_divergence(indicator_1, 1) : 0)
    array.set(divs, 1, cond and (searchdiv == 'Regular' or searchdiv == 'Regular/Hidden') ? negative_regular_negative_hidden_divergence(indicator_1, 1) : 0)
    array.set(divs, 2, cond and (searchdiv == 'Hidden' or searchdiv == 'Regular/Hidden') ? positive_regular_positive_hidden_divergence(indicator_1, 2) : 0)
    array.set(divs, 3, cond and (searchdiv == 'Hidden' or searchdiv == 'Regular/Hidden') ? negative_regular_negative_hidden_divergence(indicator_1, 2) : 0)
    divs

// array to keep all divergences
var all_divergences = array.new_int(44)  // 11 indicators * 4 divergence = 44 elements
// set related array elements
array_set_divs(div_pointer, index) =>
    for x = 0 to 3 by 1
        array.set(all_divergences, index * 4 + x, array.get(div_pointer, x))

// set divergences array 
array_set_divs(calculate_divs(calcrsi, rsid), 2)
// check minimum number of divergence, if less than showlimit then delete all divergence
total_div = 0
for x = 0 to array.size(all_divergences) - 1 by 1
    total_div += math.round(math.sign(array.get(all_divergences, x)))
    total_div

if total_div < showlimit
    array.fill(all_divergences, 0)

// keep line in an array
var pos_div_lines = array.new_line(0)
var neg_div_lines = array.new_line(0)
var pos_div_labels = array.new_label(0)
var neg_div_labels = array.new_label(0)

// remove old lines and labels if showlast option is enabled
delete_old_pos_div_lines() =>
    if array.size(pos_div_lines) > 0
        for j = 0 to array.size(pos_div_lines) - 1 by 1
            line.delete(array.get(pos_div_lines, j))
        array.clear(pos_div_lines)

delete_old_neg_div_lines() =>
    if array.size(neg_div_lines) > 0
        for j = 0 to array.size(neg_div_lines) - 1 by 1
            line.delete(array.get(neg_div_lines, j))
        array.clear(neg_div_lines)

delete_old_pos_div_labels() =>
    if array.size(pos_div_labels) > 0
        for j = 0 to array.size(pos_div_labels) - 1 by 1
            label.delete(array.get(pos_div_labels, j))
        array.clear(pos_div_labels)

delete_old_neg_div_labels() =>
    if array.size(neg_div_labels) > 0
        for j = 0 to array.size(neg_div_labels) - 1 by 1
            label.delete(array.get(neg_div_labels, j))
        array.clear(neg_div_labels)

// delete last creted lines and labels until we met new PH/PV 
delete_last_pos_div_lines_label(n) =>
    if n > 0 and array.size(pos_div_lines) >= n
        asz = array.size(pos_div_lines)
        for j = 1 to n by 1
            line.delete(array.get(pos_div_lines, asz - j))
            array.pop(pos_div_lines)
        if array.size(pos_div_labels) > 0
            label.delete(array.get(pos_div_labels, array.size(pos_div_labels) - 1))
            array.pop(pos_div_labels)

delete_last_neg_div_lines_label(n) =>
    if n > 0 and array.size(neg_div_lines) >= n
        asz = array.size(neg_div_lines)
        for j = 1 to n by 1
            line.delete(array.get(neg_div_lines, asz - j))
            array.pop(neg_div_lines)
        if array.size(neg_div_labels) > 0
            label.delete(array.get(neg_div_labels, array.size(neg_div_labels) - 1))
            array.pop(neg_div_labels)

// variables for Alerts
pos_reg_div_detected = false
neg_reg_div_detected = false
pos_hid_div_detected = false
neg_hid_div_detected = false

// to remove lines/labels until we met new // PH/PL
var last_pos_div_lines = 0
var last_neg_div_lines = 0
var remove_last_pos_divs = false
var remove_last_neg_divs = false
if pl
    remove_last_pos_divs := false
    last_pos_div_lines := 0
    last_pos_div_lines
if ph
    remove_last_neg_divs := false
    last_neg_div_lines := 0
    last_neg_div_lines

// draw divergences lines and labels
divergence_text_top = ''
divergence_text_bottom = ''
distances = array.new_int(0)
dnumdiv_top = 0
dnumdiv_bottom = 0
top_label_col = color.white
bottom_label_col = color.white
old_pos_divs_can_be_removed = true
old_neg_divs_can_be_removed = true
startpoint = dontconfirm ? 0 : 1  // used for don't confirm option

for x = 0 to 10 by 1
    div_type = -1
    for y = 0 to 3 by 1
        if array.get(all_divergences, x * 4 + y) > 0  // any divergence?
            div_type := y
            if y % 2 == 1
                dnumdiv_top += 1
                top_label_col := array.get(div_colors, y)
                top_label_col
            if y % 2 == 0
                dnumdiv_bottom += 1
                bottom_label_col := array.get(div_colors, y)
                bottom_label_col
            if not array.includes(distances, array.get(all_divergences, x * 4 + y))  // line not exist ?
                array.push(distances, array.get(all_divergences, x * 4 + y))
                new_line = showlines ? line.new(x1=bar_index - array.get(all_divergences, x * 4 + y), y1=source == 'Close' ? close[array.get(all_divergences, x * 4 + y)] : y % 2 == 0 ? low[array.get(all_divergences, x * 4 + y)] : high[array.get(all_divergences, x * 4 + y)], x2=bar_index - startpoint, y2=source == 'Close' ? close[startpoint] : y % 2 == 0 ? low[startpoint] : high[startpoint], color=array.get(div_colors, y), style=y < 2 ? reg_div_l_style : hid_div_l_style, width=y < 2 ? reg_div_l_width : hid_div_l_width) : na
                if y % 2 == 0
                    if old_pos_divs_can_be_removed
                        old_pos_divs_can_be_removed := false
                        if not showlastd and remove_last_pos_divs
                            delete_last_pos_div_lines_label(last_pos_div_lines)
                            last_pos_div_lines := 0
                            last_pos_div_lines
                        if showlastd
                            delete_old_pos_div_lines()
                    array.push(pos_div_lines, new_line)
                    last_pos_div_lines += 1
                    remove_last_pos_divs := true
                    remove_last_pos_divs

                if y % 2 == 1
                    if old_neg_divs_can_be_removed
                        old_neg_divs_can_be_removed := false
                        if not showlastd and remove_last_neg_divs
                            delete_last_neg_div_lines_label(last_neg_div_lines)
                            last_neg_div_lines := 0
                            last_neg_div_lines
                        if showlastd
                            delete_old_neg_div_lines()
                    array.push(neg_div_lines, new_line)
                    last_neg_div_lines += 1
                    remove_last_neg_divs := true
                    remove_last_neg_divs

            // set variables for alerts
            if y == 0
                pos_reg_div_detected := true
                pos_reg_div_detected
            if y == 1
                neg_reg_div_detected := true
                neg_reg_div_detected
            if y == 2
                pos_hid_div_detected := true
                pos_hid_div_detected
            if y == 3
                neg_hid_div_detected := true
                neg_hid_div_detected
    // get text for labels
    if div_type >= 0
        divergence_text_top += (div_type % 2 == 1 ? showindis != 'Don\'t Show' ? array.get(indicators_name, x) + '\n' : '' : '')
        divergence_text_bottom += (div_type % 2 == 0 ? showindis != 'Don\'t Show' ? array.get(indicators_name, x) + '\n' : '' : '')
        divergence_text_bottom


// draw labels
if showindis != 'Don\'t Show' or shownum
    if shownum and dnumdiv_top > 0
        divergence_text_top += str.tostring(dnumdiv_top)
        divergence_text_top
    if shownum and dnumdiv_bottom > 0
        divergence_text_bottom += str.tostring(dnumdiv_bottom)
        divergence_text_bottom
    if divergence_text_top != ''
        if showlastd
            delete_old_neg_div_labels()
        array.push(neg_div_labels, label.new(x=bar_index, y=math.max(high, high[1]), text=divergence_text_top, color=top_label_col, textcolor=neg_div_text_col, style=label.style_label_down))

    if divergence_text_bottom != ''
        if showlastd
            delete_old_pos_div_labels()
        array.push(pos_div_labels, label.new(x=bar_index, y=math.min(low, low[1]), text=divergence_text_bottom, color=bottom_label_col, textcolor=pos_div_text_col, style=label.style_label_up))
