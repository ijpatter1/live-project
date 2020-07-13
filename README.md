# The Tech Academy Live Project

Welcome to my Live Project repository. This project was built using the Django framework. I was tasked with building an interactive web app for managing one's
collections of things related to various hobbies, as well as API and Data Scraped content for those hobbies.

There were 3 distinct portions of the app with individual stories covering each portion: a Database Collection Manager, Data Scraping with Beautiful Soup and a Restful API interface.

The tool set for the project consisted of the following Python packages: beautifulsoup4 4.7.1; certifi 2018.11.29; chardet 3.0.4; Django 2.1.5; idna 2.8; numpy 1.16.2; pytz 2018.9; requests 2.21.0; selenium 3.141.0; soupsieve 1.7.2; urllib3 1.24.1.

# Database collection manager

INSERT GIF HERE

Specifically, my web app focused on tracking S&P 500 quarterly earnings report data. For context, the S&P 500 is an index of 500 large companies list on US stock exchanges.

I created a django model and model form for saving company data to the database and developed basic "CRUD" functionality. 

```python
class Company(models.Model):
    ticker = UpperCharField(max_length=10, unique=True)
    company = models.CharField(max_length=30)
    investor_url = models.URLField(blank=True, null=True)
    report_date = models.DateField(blank=True, null=True)
    earnings_call = models.CharField(max_length=30, blank=True, null=True)
    estimated_eps = models.DecimalField(max_digits=4, decimal_places=2, blank=True, null=True)
    last_updated = models.DateTimeField(auto_now=True)

    @classmethod
    def create(cls, value):
        company = cls(ticker=value['symbol'],
                      company=value['company'],
                      investor_url=None,
                      report_date=value['report_date'],
                      earnings_call=value['calltime'],
                      estimated_eps=value['estimate'])
        return company

    companies = models.Manager()
    
class CompanyForm(ModelForm):
    class Meta:
        model = Company
        fields = ['ticker', 'company', 'investor_url', 'report_date', 'earnings_call', 'estimated_eps']
        labels = {
            'investor_url': _('Investor Relations'),  # change label strings
            'report_date': _('Report Date'),
            'estimated_eps': _('Estimated EPS'),
            'earnings_call': _('Earnings Call Time'),

        }
        widgets = {
            'ticker': TextInput(attrs={'placeholder': 'S&P 500 Ticker'}),  # apply placeholder attributes
            'company': TextInput(attrs={'placeholder': 'Company name'}),
            'investor_url': URLInput(attrs={'placeholder': 'https://'}),
            'report_date': SelectDateWidget(),  # add date-picker
            'earnings_call': TextInput(attrs={'placeholder': 'e.g., Before Market Open'}),
            'estimated_eps': NumberInput(attrs={'placeholder': '0.00'})
        }
```

I wrote the views and templates necessary to handle requests and display the data from the database.

```python
def index(request):
    if request.method == 'POST':
        values = request.POST.getlist('checks')  # retrieves the values of all the checked checkboxes
        if values:  # if any boxes where checked
            for ticker in values:  # get the company from the database and delete it
                company = get_object_or_404(Company, ticker=ticker)
                company.delete()
            return redirect('index')
        else:
            return redirect('index')
    else:
        get_companies = Company.companies.all()  # get all items from the database
        context = {'companies': get_companies}  # package items to be sent to template
        return render(request, 'EarningsApp/earningsapp_index.html', context)


def add_company(request):  # view function to display Company ModelForm for adding companies
    if request.method == 'POST':  # if the user is submitting a form
        form = CompanyForm(request.POST)
        if form.is_valid():
            ticker = request.POST.get('ticker')
            if in_sp500(ticker):
                form.save()
                return redirect('index')  # redirect to index if form is valid
            else:
                error_message = f'{ticker} is not a component of the S&P 500'
                form = CompanyForm(request.POST)
                context = {'form': form, 'error_message': error_message}
                return render(request, 'EarningsApp/earningsapp_add.html', context)
    else:
        form = CompanyForm()  # render the form
    return render(request, 'EarningsApp/earningsapp_add.html', {'form': form})
    
def edit_company(request, ticker):
    ticker = str(ticker)
    company = get_object_or_404(Company, ticker=ticker)
    if request.method == 'POST':
        form = CompanyForm(request.POST, instance=company)
        value = request.POST.get('submit')
        if value == "Submit":
            if form.is_valid():
                tkr = request.POST.get('ticker')
                if in_sp500(tkr):
                    form.save()
                    return redirect('index')  # redirect to index if form is valid
                else:
                    error_message = f'{tkr} is not a component of the S&P 500'
                    context = {'form': form, 'error_message': error_message}
                    return render(request, 'EarningsApp/earningsapp_edit.html', context)
        if value == "Delete":
            company.delete()
            return redirect('index')
    else:
        form = CompanyForm(instance=company)
    context = {'form': form, 'ticker': ticker, 'company': company}
    return render(request, 'EarningsApp/earningsapp_edit.html', context)
```

This included a details page that featured widgets providing more in depth data.
```python
# this view renders the details page and all sub-pages linked on the details page
def company_details(request, ticker):
    ticker = str(ticker)
    company = get_object_or_404(Company, ticker=ticker)
    context = {'company': company}
    category = ticker
    stock_news = get_news(category)
    context.update(stock_news)
    if request.path == f'/EarningsApp/Index/{ticker}/Details/Chart':
        return render(request, 'EarningsApp/earningsapp_details_chart.html', context)
    else:
        return render(request, 'EarningsApp/earningsapp_details.html', context)
```

# Data Scraping with Beautiful Soup

INSERT GIF HERE

I used Beautiful Soup to scrap data from Yahoo Finance: https://finance.yahoo.com/calendar/earnings.

```python
def calendar(request):
    if request.method == 'POST':
        report_date = request.POST.get('report_date')
        report_date = datetime.datetime.strptime(report_date, '%Y-%m-%d')
        report_date_dict = {'report_date': report_date}
        values = request.POST.getlist('checks')
        if values:
            for value in values:
                value = ast.literal_eval(value)
                value.update(report_date_dict)
                try:
                    company = Company.companies.get(ticker=value['symbol'])
                    company.company = value['company']
                    company.report_date = value['report_date']
                    company.earnings_call = value['calltime']
                    company.estimated_eps = value['estimate']
                    company.save()
                except Company.DoesNotExist:
                    company = Company.create(value)
                    company.save()
        params, week_dates = get_params(report_date)
        # page request with the parameters defined above
        company_earnings = scrape_earnings(params)
        confirmation_msg = "The selected item(s) has/have been added/updated."
        context = {
            'market_time': report_date,
            'week_dates': week_dates,
            'company_earnings': company_earnings,
            'confirmation_msg': confirmation_msg
        }
        return render(request, 'EarningsApp/earningsapp_calendar.html', context)
    # declare datetime_object to be passed as an argument to get_params()
    date_obj = timezone.now()
    # if a specific date was requested along with the page, convert the string to a datetime obj
    date_str = request.GET.get('date')
    if date_str:
        date_obj = datetime.datetime.strptime(date_str, '%Y-%m-%d')
    # utility function to automatically define parameters
    params, week_dates = get_params(date_obj)
    # page request with the parameters defined above
    company_earnings = scrape_earnings(params)
    context = {'market_time': date_obj, 'week_dates': week_dates, 'company_earnings': company_earnings}
    return render(request, 'EarningsApp/earningsapp_calendar.html', context)

```

# Restful API interface

INSERT GIF HERE

I used the Yahoo finance API from rapidapi.com to request news articles relating to S&P 500 companies.
