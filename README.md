# Monthly_Calendar_View

```swift
import SwiftUI

// MARK: - MODELS, DateValue
struct DateValue: Identifiable {
    var id: String = UUID().uuidString
    var day: Int
    var date: Date
}

struct MonthView: View {
    
    var theme: Theme = UserDefaults.theme
    var font: String = UserDefaults.customFont
    
    @State var selectedDay: Date = Date()
    let onSelectedDay: (Date) -> Void
    
    // Month update on arrow button click
    @State private var selectedMonthIndex: Int = 0
    
    // Name of Day
    private let namesOfDays: [String] = Calendar.current.shortWeekdaySymbols
    
    // Days
    private let columns: [GridItem] = Array(repeating: GridItem(.flexible()), count: 7)
    
    private var currentMonth: Date {
        let calendar: Calendar = Calendar.current
        // Getting Current Month Data...
        guard let currentMonth = calendar.date(byAdding: .month, value: self.selectedMonthIndex, to: Date()) else {
            return Date()
        }
        return currentMonth
    }
    
    var body: some View {
        
        VStack(spacing: 0) {
            
            Header()
            
            // MARK: - DAY VIEW
            HStack(spacing: 0) {
                ForEach(self.namesOfDays, id: \.self) { day in
                    Text(day)
                        .foregroundColor(self.theme.backgroundColor.fontColor)
                        .adaptiveFontSize(customFont: self.font, 14)
                        .frame(maxWidth: .infinity)
                }
            } //: HSTACK - DAY VIEW
            .adaptivePadding(.bottom, 16)
            

            // MARK: - DATES
            LazyVGrid(columns: columns, spacing: 0) {
                let dateValues = self.extractDateValues()
                ForEach(dateValues) { value in
                    CardView(value: value)
                        .frame(maxHeight: .infinity, alignment: .top)
                }
            } //: LAZYVGRID DATES
            
        } //: VSTACK
        .onChange(of: self.selectedMonthIndex) { newValue in
            // updating Month...
            self.selectedDay = self.currentMonth
        }
        
    }
    
    // MARK: CHANGE DAY DESIGN IF YOU UWANT
    @ViewBuilder
    private func CardView(value: DateValue) -> some View {
        VStack {
            if value.day == -1 {
                EmptyView()
            } else if value.date.uniqueID == self.selectedDay.uniqueID {
                Text("\(value.day)")
                    .foregroundColor(self.theme.primaryColor.fontColor)
                    .adaptiveFontSize(customFont: self.font, 14)
                    .adaptivePadding(4)
                    .background(self.theme.primaryColor)
                    .clipShape(Capsule())
                    .onTapGesture {
                        FeedbackGenerator.impact()
                        withAnimation {
                            self.selectedDay = value.date
                        }
                    }
            } else {
                Text("\(value.day)")
                    .foregroundColor(self.theme.backgroundColor.fontColor)
                    .adaptiveFontSize(customFont: self.font, 14)
                    .adaptivePadding(4)
                    .onTapGesture {
                        FeedbackGenerator.impact()
                        withAnimation {
                            self.selectedDay = value.date
                        }
                    }
            }
            
      
        } //: VSTACK
        .frame(height: resized(70), alignment: .top)
        
    }
    
    private func Header() -> some View {
        HStack(spacing: 20) {
            HStack(spacing: 0) {
                Text(self.selectedDay.extract(format: "YYYY"))
                    .fontWeight(.medium)
                
                Text(" ")
                
                Text(self.selectedDay.extract(format: "MMMM"))
                    .fontWeight(.medium)
            } //: VSTACK
            .foregroundColor(self.theme.backgroundColor.fontColor)
            .adaptiveFontSize(customFont: self.font, 20)
            
            Spacer(minLength: 0)
            
            CustomButton {
                withAnimation {
                    self.selectedMonthIndex -= 1
                }
            } label: {
                Image(systemName: "chevron.left")
                    .foregroundColor(self.theme.primaryColor)
                    .adaptiveFontSize(24)
            }
            
            CustomButton {
                withAnimation {
                    self.selectedMonthIndex += 1
                }
            } label: {
                Image(systemName: "chevron.right")
                    .foregroundColor(self.theme.primaryColor)
                    .adaptiveFontSize(24)
            }
            
        } //: HSTACK - HEADER
        .adaptivePadding(.bottom, 16)
    }

}

// MARK: METHODS
extension MonthView {
    
    // with `currentMonth`, extract relevant dates
    private func extractDateValues() -> [DateValue] {
        let calendar: Calendar = Calendar.current
        // Getting Current Month Data...
        let currentMonth: Date = self.currentMonth
        var days: [DateValue] = currentMonth.allDaysOfTheMonth.compactMap { date -> DateValue in
            // getting days...
            let day: Int = calendar.component(.day, from: date)
            return DateValue(day: day, date: date)
        }
        // MARK: YOU CAN FIX OFFSET DAYS HERE
        // adding offset days to get exact week day...
        let firstWeekDay = calendar.component(.weekday, from: days.first!.date)
        for _ in 0..<firstWeekDay - 1 {
            days.insert(DateValue(day: -1, date: Date()), at: 0)
        }
        return days
    }
    
}

// Extending Date to get Current Month Dates...
fileprivate extension Date {
    
    var allDaysOfTheMonth: [Date] {
        let calendar: Calendar = Calendar.current
        
        // getting start Date...
        let dateComponents: DateComponents = calendar.dateComponents([.year, .month], from: self)
        let startDate: Date = calendar.date(from: dateComponents)!
        
        let range: Range<Int> = calendar.range(of: .day, in: .month, for: startDate)!
        
        // getting date...
        return range.compactMap { day -> Date in
            return calendar.date(byAdding: .day, value: day - 1, to: startDate)!
        }
    }
    
    var uniqueID: String {
        let calendarDate: DateComponents = Calendar.current.dateComponents([.day, .year, .month], from: self)
        return "\(calendarDate.month!)-\(calendarDate.day!)-\(calendarDate.year!)"
    }
    
}



#if DEBUG
struct CustomDatePicker_Previews: PreviewProvider {
    static var previews: some View {
        ZStack {
            Theme.purpleDark.backgroundColor.ignoresSafeArea()
            MonthView(theme: .purpleDark, onSelectedDay: { date in
                
            })
        }
    }
}
#endif

```
