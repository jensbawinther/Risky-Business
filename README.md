# Risky-Business
Risky Business. Code, Materials, talks, paper. Developed by Jesper Lyng Jensen 
The code as follows:

#Iterations of rb2 - output after 9 turn in excel and statestic. Run min. 36 project to fill up boarimport numpy as np
import pandas as pd
import random
import itertools
import numpy as np
from collections import Counter
from google.colab import files

# Set output flag: 1 for export til excelfile, 0 for no export
output = 1
# DataFrames to store all runs
df_saldo_ops = pd.DataFrame()
df_k5 = pd.DataFrame()

output_file_path = '/content/rb2_output.xlsx'

# Arrays to store results for 1000 iterations
k5_at_turn_10_array = [] # we will use a 9 turn array.
saldo_at_turn_10_array = [] # we will use a 9 turn array

k5_at_turn_9_array = [] # this is for the 9 turn array
saldo_at_turn_9_array = [] # this is for the 9 turn array

#Miximum numbers of iterations. Change the number here
max_iterations = 1000 # min 25

# Maximun numbers of projects.
max_projects = 5


# Limitation for initialtion a new project. When we run project-1 we set saldo_limit to 19
cash_limit = 19


count = 0 # Run iterations and count

for iteration in range(max_iterations):
  # Load the data from the Excel file
  # Initialize k5_turn and saldo_ops_turn for this iteration
  k5_turn = {}  # Add this line to initialize k5_turn
  saldo_ops_turn = {}# add this line inititialize
  file_path = 'rb2_48projects.xlsx'
  df_board = pd.read_excel(file_path, sheet_name='board', index_col=0)
  df_project = pd.read_excel(file_path, sheet_name='projects', index_col=0)
  df_raw_optimizing = pd.DataFrame(columns=['Turn', 'Project','sum_existing_sunk_ops','current_sunk','current_ops','sum_exist_sunk_ops_c_ops','sum_sunk_project'])
  df_combinations = pd.DataFrame(columns=['Turn', 'Projects', 'Total sum_sunk_project', 'Total current_ops'])

  # Fill NaN values with 0
  df_board.fillna(0, inplace=True)
  df_project.fillna(0, inplace=True)

  # Variables from the board
  pharmabugs_1_turn_max = df_board.at['pharmabugs', '1_turn']
  pharmabugs_2_turn_max = df_board.at['pharmabugs', '2_turn']
  pharmabugs_3_turn_max = df_board.at['pharmabugs', '3_turn']
  pharmabugs_4_turn_max = df_board.at['pharmabugs', '4_turn']
  pharmabugs_5_turn_max = df_board.at['pharmabugs', '5_turn']
  pharmabugs_6_turn_max = df_board.at['pharmabugs', '6_turn']
  pharmabugs_7_turn_max = df_board.at['pharmabugs', '7_turn']
  pharmabugs_8_turn_max = df_board.at['pharmabugs', '8_turn']
  pharmabugs_9_turn_max = df_board.at['pharmabugs', '9_turn']
  pharmabugs_10_turn_max = df_board.at['pharmabugs', '10_turn']


  # Initialize saldo variables for each turn
  saldo_1_turn = pharmabugs_1_turn_max
  saldo_2_turn = pharmabugs_2_turn_max
  saldo_3_turn = pharmabugs_3_turn_max
  saldo_4_turn = pharmabugs_4_turn_max
  saldo_5_turn = pharmabugs_5_turn_max
  saldo_6_turn = pharmabugs_6_turn_max
  saldo_7_turn = pharmabugs_7_turn_max
  saldo_8_turn = pharmabugs_8_turn_max
  saldo_9_turn = pharmabugs_9_turn_max
  saldo_10_turn = pharmabugs_10_turn_max

  # Dictionary to map turns to their corresponding max values
  pharmabugs_turns = {
      1: pharmabugs_1_turn_max,
      2: pharmabugs_2_turn_max,
      3: pharmabugs_3_turn_max,
      4: pharmabugs_4_turn_max,
      5: pharmabugs_5_turn_max,
      6: pharmabugs_6_turn_max,
      7: pharmabugs_7_turn_max,
      8: pharmabugs_8_turn_max,
      9: pharmabugs_9_turn_max,
      10: pharmabugs_10_turn_max
  }
  # var saldo_ops
  sal_ops_1_turn = 0
  sal_ops_2_turn = 0
  sal_ops_3_turn = 0
  sal_ops_4_turn = 0
  sal_ops_5_turn = 0
  sal_ops_6_turn = 0
  sal_ops_7_turn = 0
  sal_ops_8_turn = 0
  sal_ops_9_turn = 0
  sal_ops_10_turn = 0
  #  dict to map turn to their saldo_ops
  saldo_ops_turn = {
      1: sal_ops_1_turn,
      2: sal_ops_2_turn,
      3: sal_ops_3_turn,
      4: sal_ops_4_turn,
      5: sal_ops_5_turn,
      6: sal_ops_6_turn,
      7: sal_ops_7_turn,
      8: sal_ops_8_turn,
      9: sal_ops_9_turn,
      10: sal_ops_10_turn
  }

  count += 1

  # Initialize saldo_ops variable, K4
  saldo_ops_previous = 0.0
  K4_previous = 0.0


  # Track saldo for each turn
  saldo_turns = {}

  # List of possible values for random number generation for lib random
  values_036 = [0, 3, 6]
  values_048 = [0, 4, 8] # for testing of high risk
  # Interest rate
  interest_value = 0.5

  # List of project names
  project_names = df_project.index.tolist()

  # Initialize K3_value_previous variables
  K3_value_previous = 0.0
  K5_total = 0.0

  # Dictionary to store the turn when each project was added
  turn_added_dict = {}

  # Calculate the project surplus for each project. This surplus is calculated at the end of each turn and is therefore obsolite
  for project in project_names:
      # Sum all _ops and _sunk values across all turns for the project
      ops_sunk_sum = 0
      for col in df_project.columns:
          if '_ops' in col or '_sunk' in col:
              ops_sunk_sum += df_project.at[project, col]

      # Calculate surplus with interest rate
      project_surplus = ops_sunk_sum * (1 + interest_value)

      # Add surplus to the df_project as a new column
      df_project.at[project, 'project_surplus'] = project_surplus

  # Create variables for each project surplus
  project_surpluses = {project: df_project.at[project, 'project_surplus'] for project in project_names}

  # Function to add project to the board
  def add_project_to_board(df_board, df_project, project_name, start_pos):
      project_row = df_project.loc[project_name]

      # Separate the project row into sunk, ops, and prob
      sunk_values = project_row.filter(like='_sunk').values
      ops_values = project_row.filter(like='_ops').values
      prob_values = project_row.filter(like='_prob').values

      # Calculate the required positions
      sunk_start_pos = start_pos - 1
      ops_start_pos = start_pos - 1
      prob_start_pos = start_pos - 1

      # Prepare rows for new data
      new_sunk_row = [0] * len(df_board.columns)
      new_ops_row = [0] * len(df_board.columns)
      new_prob_row = [0] * len(df_board.columns)

      # Fill the rows with the project data, ensuring they fit the board's column count
      new_sunk_row[sunk_start_pos:sunk_start_pos+len(sunk_values)] = sunk_values[:len(df_board.columns) - sunk_start_pos]
      new_ops_row[ops_start_pos:ops_start_pos+len(ops_values)] = ops_values[:len(df_board.columns) - ops_start_pos]
      new_prob_row[prob_start_pos:prob_start_pos+len(prob_values)] = prob_values[:len(df_board.columns) - prob_start_pos]

      # Add the new rows to the board DataFrame
      df_board.loc[f'{project_name}_sunk'] = new_sunk_row
      df_board.loc[f'{project_name}_ops'] = new_ops_row
      df_board.loc[f'{project_name}_prob'] = new_prob_row

      return df_board


  # Main loop through each turn
  for turn in range(1, 11):
      # Calculate the sum of existing sunk and ops for all projects in the current turn
      existing_sunk = sum(df_board.at[f'{project}_sunk', f'{turn}_turn'] for project in project_names if f'{project}_sunk' in df_board.index)
      existing_ops = sum(df_board.at[f'{project}_ops', f'{turn}_turn'] for project in project_names if f'{project}_ops' in df_board.index)
      saldo = pharmabugs_turns[turn]
      turn_ops = turn
      turn_original = turn
      #print(f'initial saldo{saldo}')
      individual_projects_list = []
      total_ops_sum_removel = 0.0 # hold the ops total for the turn
      # Subtract existing projects costs
      saldo -= (existing_sunk + existing_ops)
      #print(f"Turn {turn}: Existing Sunk = {existing_sunk}, Existing Ops = {existing_ops}, Saldo = {saldo}")
      # Store the initial saldo for this turn
      saldo_turns[turn] = saldo

      #print(f"Initial Saldo for Turn {turn}: {saldo}")

      # Extract project names from df_board by filtering '_sunk' in index
      board_projects = [index.split('_sunk')[0] for index in df_board.index if '_sunk' in index]
      #print(f"board project: {board_projects}")

      # Filter projects that have not been added yet
      available_projects = [project for project in project_names if f'{project}_sunk' not in df_board.index]
      #print(f"Available Projects for Turn {turn}: {available_projects}")
      #print(project_names)
      # Check for inactive projects and remove them from the count
      inactive_projects = []
      for project in project_names:
          if f'{project}_sunk' in df_board.index:
              if turn > 1:  # Ensure it's not the first turn
                  # Check if the project is inactive (zero in the current and previous turns)
                  if (df_board.at[f'{project}_ops', f'{turn-1}_turn'] > 0 or df_board.at[f'{project}_sunk', f'{turn-1}_turn'] > 0) and \
                    (df_board.at[f'{project}_ops', f'{turn}_turn'] == 0 and df_board.at[f'{project}_sunk', f'{turn}_turn'] == 0):
                      inactive_projects.append(project)
                      # Add the surplus of the inactive project to the saldo_ops for this turn
                      #project_surplus = df_project.at[project, 'project_surplus']
                      #saldo_ops_previous += project_surplus  # Add the surplus to saldo_ops
                      #print(f"Project {project} became inactive on Turn {turn}. Not Added surplus {project_surplus} to saldo_ops. inactive projects: {inactive_projects}")

                  if (df_board.at[f'{project}_ops', f'{turn}_turn'] == 0 and df_board.at[f'{project}_sunk', f'{turn}_turn'] == 0):
                      inactive_projects.append(project)
                      #print(f"Project {project} became inactive before Turn {turn}.")

      # Remove inactive projects from the count of existing projects
      active_projects_count = len([project for project in project_names if f'{project}_sunk' in df_board.index and project not in inactive_projects])
      #print(f"Active Projects Count for Turn {turn}: {active_projects_count} df_board.index: {df_board.index}")

      # Try to add up to 5 projects or typed in the variable max projects
      for _ in range(max_projects): # this can be a varible and so can the saldo limit in the next line and the if sentence 5 lines below
          if saldo < cash_limit or not available_projects:
              break
          # Count the number of projects already added
          existing_projects_count = len([project for project in project_names if f'{project}_sunk' in df_board.index])

          if saldo < cash_limit or not available_projects or active_projects_count >= max_projects:
              print(f"Cannot add more projects on Turn {turn} because the limit of 5 active projects has been reached.")
              break

          # Randomly select an available project
          selected_project = random.choice(available_projects)
          available_projects.remove(selected_project)  # Remove from available list

        # Check if this project is being added for the first time
          if selected_project not in turn_added_dict:
              # Store the turn when the project is first added
              turn_added_dict[selected_project] = turn
              #print(f"Project {selected_project} was added on Turn {turn} ")


          # Calculate the impact of adding this project
          turn_added = turn_added_dict[selected_project]
          turn_index = turn - turn_added + 1

          project_sunk = df_project.at[selected_project, f'{turn_index}_sunk'] # change turn to turn_index
          project_ops = df_project.at[selected_project, f'{turn_index}_ops']
          #print(f"project_sunk {project_sunk} project_ops {project_ops} turn_index: {turn_index}")
          project_sunk_test2 = df_project.at[selected_project, f'{turn_index}_sunk']
          project_ops_test2 = df_project.at[selected_project, f'{turn_index}_ops']


          if project_sunk == 0 and project_ops == 0: # finds the value in df_board

            # Make sure the turn index doesn't go out of bounds
            # Assuming `turn_added` is the turn the project was added the real code is turn_index = turn - turn_added
            turn_added = turn_added_dict[selected_project]
            turn_index = turn - turn_added


            if turn_index >= 0 and turn_index < len(df_project.columns):
              project_sunk = df_project.at[selected_project, f'{turn_added + turn_index}_sunk']
              project_ops = df_project.at[selected_project, f'{turn_added + turn_index}_ops']

            else:
              print(f"Turn index {turn_index} is out of bounds for project {selected_project}")

            #print(f"Selected Project: {selected_project}")
            #print(f"Turn: {turn}")
            #print(f"Index in df_board: {df_board.index}")

          #print(f"project_sunk {project_sunk}")
          #print(f"project_ops {project_ops}")


          # New saldo after adding this project
          new_saldo = saldo - (project_sunk + project_ops)
          #print(f"print new saldo before if {new_saldo}")

          if new_saldo >= 0: # remember here new_saldo must cover the cost of the project, not still another project => 0
              # Add project to the board
              df_board = add_project_to_board(df_board, df_project, selected_project, turn)
              saldo = new_saldo  # Update saldo
              df_board.at['saldo', f'{turn}_turn'] = saldo  # Update saldo in df_board
              #print(f"Added {selected_project} on Turn {turn}. New Saldo: {saldo}")
              # Extract project names from df_board by filtering '_sunk' in index again
              board_projects = [index.split('_sunk')[0] for index in df_board.index if '_sunk' in index]
              #print(f"board project after adding project: {board_projects}")
          else:
              print(f"Cannot add {selected_project} on Turn {turn} because saldo: {saldo} would drop below 10.")
              break
      #print(saldo)
      # Store the final saldo for this turn
      if turn == 1:
          saldo_1_turn = saldo
      elif turn == 2:
          saldo_2_turn = saldo
      elif turn == 3:
          saldo_3_turn = saldo
      elif turn == 4:
          saldo_4_turn = saldo
      elif turn == 5:
          saldo_5_turn = saldo
      elif turn == 6:
          saldo_6_turn = saldo
      elif turn == 7:
          saldo_7_turn = saldo
      elif turn == 8:
          saldo_8_turn = saldo
      elif turn == 9:
          saldo_9_turn = saldo
      elif turn == 10:
          saldo_10_turn = saldo

      #K3_value calculated
      existing_sunk_allprojects = sum(df_board.at[f'{project}_sunk', f'{turn}_turn'] for project in project_names if f'{project}_sunk' in df_board.index)
      existing_ops_allprojects = sum(df_board.at[f'{project}_ops', f'{turn}_turn'] for project in project_names if f'{project}_ops' in df_board.index)
      K3_value = (existing_ops_allprojects + existing_sunk_allprojects) * (1 + interest_value)
      df_board.at['K3_value', f'{turn_ops}_turn'] = K3_value
      #print(f"K3_value: {K3_value}")

      # MAX RISK SIM. df_board for project if (project_{project}_sunk >< 0 or project_{project}_ops <> 0) and project_{project}_prob <> 0
      #K2 take a random value in values_036 and times _prob put the result in _prob. For alle projekter put i liste. add til risk_element dict. saldo - values in risk_elemnent dict < 0 else saldo -= risk_element values
      # MAX RISK SIM. df_board for project if (project_{project}_sunk >< 0 or project_{project}_ops <> 0) and project_{project}_prob <> 0
      risk_element = {}
      for project in board_projects:
          # Check if the project is active (i.e., has non-zero sunk or ops costs)
          if df_board.at[f'{project}_sunk', f'{turn}_turn'] > 0 or df_board.at[f'{project}_ops', f'{turn}_turn'] > 0:
              project_prob = df_board.at[f'{project}_prob', f'{turn}_turn']
              risk_value = random.choice(values_036) * project_prob

              risk_element[project] = risk_value
              # Add the risk_value to the _prob column for this project and turn
              df_board.at[f'{project}_prob', f'{turn}_turn'] += risk_value
              #print(f"Risk applied for {project} on Turn {turn}: Risk Value = {risk_value}")
              df_board.at[f'K3_Risk_Value', f'{turn}_turn'] += risk_value #commentet out is it a good idea to have the risk elemtn om k3 _value
      #print(f"Risk Element: {risk_element}")

      # Sum all risk elements for the current turn
      sum_of_risk_elements = sum(risk_element.values())
      #print(f"Sum of risk elements for Turn {turn}: {sum_of_risk_elements}")

      # Calculate K2 and K2 > saldo. meaning the saldo can pay for the risk element
      K2 = saldo - sum_of_risk_elements
      #print(f"K2 for Turn {turn}: {K2}")
      if K2 >=0:
        saldo = saldo - sum_of_risk_elements

      # Add K2 to the df_board DataFrame
      df_board.at['K2', f'{turn}_turn'] = K2
      # -
      sum_of_risk_elements
      df_board.at['K2', f'{turn}_turn'] = K2

      # Create DataFrame if K2 is less than zero
      # Populate df_raw_optimizing if K2
      if K2 < 0:

          for project in board_projects:
              if project in turn_added_dict and (df_board.at[f'{project}_sunk', f'{turn}_turn'] > 0 or df_board.at[f'{project}_ops', f'{turn}_turn'] > 0):
                  turn_added = turn_added_dict[project]
                  #print(f"Project {project} was added on Turn {turn_added}, var turn_added.")


                  # Sum of existing _sunk and _ops from start to current turn
                  sum_existing_sunk_ops = sum(df_board.at[f'{project}_sunk', f'{t}_turn'] + df_board.at[f'{project}_ops', f'{t}_turn'] for t in range(turn_added, turn+1))

                  # Current _sunk and _ops for the project in this turn
                  current_sunk = df_board.at[f'{project}_sunk', f'{turn}_turn']
                  current_ops = df_board.at[f'{project}_ops', f'{turn}_turn']
                  #print(f"Current Sunk and Ops for {project} in Turn {turn}: Sunk = {current_sunk}, Ops = {current_ops}")
                  # Sum of existing _sunk and _ops minus current _ops
                  sum_exist_sunk_ops_c_ops = sum_existing_sunk_ops - current_ops

                  # Calculate sum_sunk_project
                  sum_sunk_project = (sum_existing_sunk_ops - current_sunk - current_ops) * (1 + interest_value) + current_sunk


                  # Use loc to add the calculated values to the df_raw_optimizing DataFrame
                  df_raw_optimizing.loc[len(df_raw_optimizing)] = {
                      'Turn': turn,
                      'Project': project,
                      'sum_existing_sunk_ops': sum_existing_sunk_ops,
                      'current_sunk': current_sunk,
                      'current_ops': current_ops,
                      'sum_exist_sunk_ops_c_ops': sum_exist_sunk_ops_c_ops,
                      'sum_sunk_project': sum_sunk_project
                  }

          #print(f"df raw optimazing : {df_raw_optimizing}")

          # Initialize the row index counter
          row_index = 0

          # Group df_raw_optimizing by Turn to calculate combinations per turn
          for turn, turn_group in df_raw_optimizing.groupby('Turn'):
              # Generate all possible combinations of projects in the current turn
              for r in range(1, len(turn_group) + 1):

                  combos = list(itertools.combinations(turn_group.itertuples(index=False), r))
                  for combo in combos:
                      projects_combined = ' & '.join([project.Project for project in combo])
                      total_sum_sunk_project = sum([project.sum_sunk_project for project in combo])
                      total_current_ops = sum([project.current_ops for project in combo])

                      # Use loc[] to add a new row to df_combinations
                      df_combinations.loc[row_index] = {
                          'Turn': turn,
                          'Projects': projects_combined,
                          'Total sum_sunk_project': total_sum_sunk_project,
                          'Total current_ops': total_current_ops
                      }
                      row_index += 1

          # Display the combinations DataFrame
          # Filter out all rows not related to current turns before the loop
          df_combinations_filtered = df_combinations[df_combinations['Turn'].isin(range(1, 11))].copy()

          #print(f"df_combinations printed: {df_combinations}")
          #print(f"turn before combinations {turn}")


          # Sort df_combinations by Total sum_sunk_project in descending order
          df_combinations_filtered = df_combinations_filtered.sort_values(by='Total sum_sunk_project', ascending=False)
          # Selecting the best combination based on K2 + Total current_ops >= 0
          selected_combinations = []

          for turn_combination in range(1, 11): # change turn to turn_combination only varible used in this for loop
              #print(saldo)

              # Create a dictionary that holds the saldo for each turn
              # Filter valid combinations where Total current_ops + saldo >= 0
              # Use .loc[] to properly filter with matching indices
              valid_combinations = df_combinations_filtered.loc[
              (df_combinations_filtered['Turn'] == turn_ops) &
              (df_combinations_filtered['Total current_ops'] + K2 >= 0) # added K2 insteed of saldo
          ]

              #print(f"df combinations filtered: {df_combinations_filtered}")
              #print(f"valid combinations: {valid_combinations}")
              individual_projects_list.clear()
              if not valid_combinations.empty:
                  # Find the combination with the minimum sum_sunk_project
                  best_combination = valid_combinations.nsmallest(1, 'Total sum_sunk_project').iloc[0]
                  K3 = best_combination['Total current_ops'] + saldo
                  K2_discount = best_combination['Total current_ops']
                  df_board.at['K3', f'{turn}_turn'] = K3
                  df_board.at['K2_discount', f'{turn}_turn'] = K2_discount
                  saldo = K2 + K2_discount
                  #print(f"K3: {K3}")

                  # Add the selected combination to the list
                  selected_combinations.append({
                    'Turn': turn_combination,
                    'Projects': best_combination['Projects'],
                    'sum_sunk_project': best_combination['Total sum_sunk_project'],
                    'K3': K3
              })
                  # Split the selected combination into individual projects and store them in a list
                  individual_projects_list = best_combination['Projects'].split(" & ")
                  #print(f"Individual projects for removel 452: {individual_projects_list}")

              else:
                  # If no valid combinations are found, include all projects from df_combinations_filtered for the current turn
                  all_projects_in_turn = df_combinations_filtered[df_combinations_filtered['Turn'] == turn_ops]

                  # Combine all projects into a single string and remove duplicates
                  all_projects = ' & '.join(all_projects_in_turn['Projects'].unique())
                  #print(f"All projects for removel: {all_projects}")
                  # Extract individual projects, remove duplicates by converting the list to a set, then back to a list
                  individual_projects_list = list(set(itertools.chain.from_iterable([proj.split(' & ') for proj in all_projects_in_turn['Projects']])))
                  #print(f"Individual projects for removel (else block): {individual_projects_list}")
                  # Extract individual projects, remove duplicates, and filter out inactive ones at the current turn
                  individual_projects_list = [
                      project
                      for project in set(itertools.chain.from_iterable([proj.split(' & ') for proj in all_projects_in_turn['Projects']]))
                      if df_board.at[f'{project}_ops', f'{turn_ops}_turn'] > 0 or
                        df_board.at[f'{project}_sunk', f'{turn_ops}_turn'] > 0 or
                        df_board.at[f'{project}_prob', f'{turn_ops}_turn'] > 0
                  ]
                  #print(f"Filtered individual projects for removal inactive (else block): {individual_projects_list}")


                  # Now calculate total_sum_sunk_project and total_current_ops for the best combination (all projects)
                  total_sum_sunk_project = 0.0
                  total_current_ops = 0.0
                  #total_sum_sunk_project = all_projects_in_turn['Total sum_sunk_project'].sum()
                  total_current_ops = all_projects_in_turn['Total current_ops'].sum()
                  # Calculate total sum for each project in individual_projects_list
                  for project in individual_projects_list:
                      # Sum the sunk and ops values up to the current turn for the given project
                      sum_existing_sunk_ops = sum(df_board.at[f'{project}_sunk', f'{turn}_turn'] + df_board.at[f'{project}_ops', f'{turn}_turn'] for turn in range(1, turn_ops + 1))


                      # Apply a one-time interest rate adjustment
                      project_sum_with_interest = sum_existing_sunk_ops * (1 + interest_value)

                      # Add this project's sum to the total
                      total_sum_sunk_project += project_sum_with_interest

                  # Print or use the selected projects and totals
                  #print(f"Selected all projects for Turn (individuel project else statement all del) {turn}: {individual_projects_list}")
                  #print(f"Total sum_sunk_project (individuel project else statement all del 494): {total_sum_sunk_project}, Total current_ops: {total_current_ops}")

                  # Calculate K3 and update the board
                  K3 = total_current_ops + K2
                  selected_combinations.append({
                      'Turn': turn_combination,
                      'Projects': ' & '.join(individual_projects_list),  # Ensure this is properly formatted
                      'Total sum_sunk_project': total_sum_sunk_project,
                      'K3': K3
                  })
                  df_board.at['K3', f'{turn}_turn'] = K3
                  if K3 > 0: # the saldo is negative after removing all project the saldo is set to 0
                    saldo = 0
                  # Ensure each project gets added to the individual_projects_list and removed correctly
                  #print(f"Removing the following individual projects (in else statement): {individual_projects_list}")

              # ----After IF-ELSE BLOCK ----
              # ---- INSERT UPDATED SNIPPET HERE ----

          if individual_projects_list:  # Ensure we have projects to process
              print(f"Processing for if individual... and Iteration {count}")

              # Iterate over individual projects and calculate removal or other processing. change from actual to removing with interest, since added wtih interest 261024
              for project_removed_value in individual_projects_list:
                  if f'{project_removed_value}_sunk' in df_board.index and f'{project_removed_value}_ops' in df_board.index:
                      current_sunk = df_board.at[f'{project_removed_value}_sunk', f'{turn}_turn']
                      current_ops = df_board.at[f'{project_removed_value}_ops', f'{turn}_turn']
                      total_ops_sum_removel += current_ops
                      # Add to K3_removed_value in df_board
                      #df_board.at['K3_removed_value', f'{turn}_turn'] += current_sunk
                      df_board.at['K3_removed_value', f'{turn}_turn'] += ((current_sunk+current_ops) * (1 + interest_value))
                      #print(f"K3_removed_value for l517 {project_removed_value} in Turn {turn}: {df_board.at['K3_removed_value', f'{turn}_turn']}")
                      df_board.at['K2_discount', f'{turn}_turn'] += current_ops


                      # Now handle all previous turns and add (_ops + _sunk) * 1.5 to K3_removed_value
                      for prev_turn in range(1, turn_ops):  # Assuming _sunk is handled in current_sunk for current_turn  the original turn_ops + 1is changed to turn_ops
                          prev_ops = df_board.at[f'{project_removed_value}_ops', f'{prev_turn}_turn']
                          prev_sunk = df_board.at[f'{project_removed_value}_sunk', f'{prev_turn}_turn']
                          total_ops_sunk = (prev_ops + prev_sunk) * 1.5

                          # Add to K3_removed_value for the current turn
                          df_board.at['K3_removed_value', f'{prev_turn}_turn'] += total_ops_sunk
                          #print(f"K3_removed_value for l517 {project_removed_value} in Turn {prev_turn}: {df_board.at['K3_removed_value', f'{prev_turn}_turn']}")

              # Reset values of _ops, _sunk, and _prob to 0.0 for future turns
              for project_removed_value in individual_projects_list:
                  if f'{project_removed_value}_ops' in df_board.index and f'{project_removed_value}_sunk' in df_board.index:
                      for future_turn in range(turn + 1, 11):
                          df_board.at[f'{project_removed_value}_ops', f'{future_turn}_turn'] = 0.0
                          df_board.at[f'{project_removed_value}_sunk', f'{future_turn}_turn'] = 0.0
                          df_board.at[f'{project_removed_value}_prob', f'{future_turn}_turn'] = 0.0
                          #print(f"Set _ops, _sunk, and _prob to 0.0 for {project_removed_value} in Turn {future_turn}.")


          # Convert to DataFrame and display selected combinations
          df_selected_combinations = pd.DataFrame(selected_combinations)
          #print("\nSelected Project Combinations for Each Turn:")
          #print(f"df for selected combinations: {df_selected_combinations}")
          #print(f"turn before if check: {turn}")

      # Store the final saldo for this turn
      if turn == 1:
          saldo_1_turn = saldo
      elif turn == 2:
          saldo_2_turn = saldo
      elif turn == 3:
          saldo_3_turn = saldo
      elif turn == 4:
          saldo_4_turn = saldo
      elif turn == 5:
          saldo_5_turn = saldo
      elif turn == 6:
          saldo_6_turn = saldo
      elif turn == 7:
          saldo_7_turn = saldo
      elif turn == 8:
          saldo_8_turn = saldo
      elif turn == 9:
          saldo_9_turn = saldo
      elif turn == 10:
          saldo_10_turn = saldo


      # Update df_board with Saldo Variables
      df_board.at['saldo', '1_turn'] = saldo_1_turn
      df_board.at['saldo', '2_turn'] = saldo_2_turn
      df_board.at['saldo', '3_turn'] = saldo_3_turn
      df_board.at['saldo', '4_turn'] = saldo_4_turn
      df_board.at['saldo', '5_turn'] = saldo_5_turn
      df_board.at['saldo', '6_turn'] = saldo_6_turn
      df_board.at['saldo', '7_turn'] = saldo_7_turn
      df_board.at['saldo', '8_turn'] = saldo_8_turn
      df_board.at['saldo', '9_turn'] = saldo_9_turn
      df_board.at['saldo', '10_turn'] = saldo_10_turn

      saldo_turns[turn] = saldo
      #print(f"Final Saldo for Turn {turn}: {saldo}")


      # Store the initial saldo for this turn
      saldo_turns[turn] = saldo

      # calculate saldo_ops
      saldo_ops = saldo_ops_previous + saldo
      #print(f"saldo_ops last: {saldo_ops}")
      df_board.at['saldo_ops', f'{turn_ops}_turn'] = saldo_ops
      #print(f"saldo_ops from df_board: {df_board.at['saldo_ops', f'{turn_ops}_turn']}")
      # Calculate saldo_ops for the current turn
      saldo_ops = saldo_ops_previous + saldo
      saldo_ops_turn[turn_ops] = saldo_ops  # Update the saldo_ops for this turn in the dictionary

      # calculate saldo_ops
      saldo_ops = saldo_ops_previous + saldo
      df_board.at['saldo_ops', f'{turn_ops}_turn'] = saldo_ops
      # Update the saldo_ops_previous for the next turn. must be the last thing
      saldo_ops_previous = saldo_ops

      K4 = K4_previous + K3_value
      df_board.at['K4', f'{turn_ops}_turn'] = K4
      #print(f"K4: {K4}")
      K4_previous = K4

      # K5 = K4 - df_board.at['K3_removed_value',f'{turn_ops}_turn']
      #df_board.at['K5', f'{turn_ops}_turn'] = K5
      #print(f"K5: K5 {df_board.at['K3_removed_value',f'{turn_ops}_turn']} {turn_ops}_turn")
      #print(f'Total ops sum removel: {total_ops_sum_removel}')
      if total_ops_sum_removel != 0:
        saldo = K2 + total_ops_sum_removel

      #print("-" * 40)
  # Calculate K5 for each turn as K5 = K4 - K3_removed_value and store in df_board

  for turn in range(1, 11):  # Loop through each turn (1 to 10)
      # Check if the K4 and K3_removed_value are present and valid for the turn
      if f'{turn}_turn' in df_board.columns:
          K4_value = df_board.at['K4', f'{turn}_turn']
          K3_removed_value = df_board.at['K3_removed_value', f'{turn}_turn']
          K3_value = df_board.at['K3_value', f'{turn}_turn']
          # Ensure both values are numbers (handle NaN or missing values)
          if pd.isna(K4_value):
              K4_value = 0
          if pd.isna(K3_removed_value):
              K3_removed_value = 0

          # Calculate K5 as K4 - K3_removed_value
          #K5_value = K4_value - K3_removed_value
          K5_value = K3_value - K3_removed_value
          K5_total += K5_value
          # Store K5 in df_board for the current turn
          df_board.at['K5', f'{turn}_turn'] = K5_total

          #print(f"Turn {turn}: K4 = {K4_value}, K3_removed_value = {K3_removed_value}, K5 = {K5_value}")
      K3_value_previous = K3_value
      k5_turn[turn] = df_board.at['K5', f'{turn}_turn']
      saldo_ops_turn[turn] = df_board.at['saldo_ops', f'{turn}_turn']
  # Output the Final Board State and Saldos
  #print("\nFinal Board State:")
  #print(df_board)
  #print("\nFinal Saldos per Turn:")
  #for turn in range(1, 11):
      #print(f"Turn {turn}: Saldo = {saldo_turns[turn]}")
  # result
  #print(f"cash, value: {df_board.at['saldo_ops','10_turn']} {df_board.at['K4', '10_turn']} ")

# After the main loop, store the values for K5 and saldo at turn 10
  k5_at_turn_9 = df_board.at['K5', '9_turn']
  saldo_at_turn_9 = df_board.at['saldo_ops', '9_turn']

  k5_at_turn_9_array.append(k5_at_turn_9)
  saldo_at_turn_9_array.append(saldo_at_turn_9)


  # Add this iteration's values to the summary DataFrames
  df_saldo_ops = pd.concat([df_saldo_ops, pd.DataFrame(saldo_ops_turn, index=[f'Run_{iteration+1}'])])
  df_k5 = pd.concat([df_k5, pd.DataFrame(k5_turn, index=[f'Run_{iteration+1}'])])

# Calculate statistics for K5 at Turn 9
average_k5_at_turn_9 = np.mean(k5_at_turn_9_array)
std_k5_at_turn_9 = np.std(k5_at_turn_9_array)
min_k5_at_turn_9 = np.min(k5_at_turn_9_array)
max_k5_at_turn_9 = np.max(k5_at_turn_9_array)
median_k5_at_turn_9 = np.median(k5_at_turn_9_array)
most_common_k5_at_turn_9 = Counter(k5_at_turn_9_array).most_common(5)
first_5_k5_at_turn_9 = k5_at_turn_9_array[:5]

# Calculate statistics for Saldo Ops at Turn 10
average_saldo_at_turn_9 = np.mean(saldo_at_turn_9_array)
std_saldo_at_turn_9 = np.std(saldo_at_turn_9_array)
min_saldo_at_turn_9 = np.min(saldo_at_turn_9_array)
max_saldo_at_turn_9 = np.max(saldo_at_turn_9_array)
median_saldo_at_turn_9 = np.median(saldo_at_turn_9_array)
most_common_saldo_at_turn_9 = Counter(saldo_at_turn_9_array).most_common(5)
first_5_saldo_at_turn_9 = saldo_at_turn_9_array[:5]

# Print the results for K5 at Turn 10
print(f" Iteration: {max_iterations}, max saldo limit: {cash_limit}, max project: {max_projects}")
print(f"Average K5: {average_k5_at_turn_9}")
print(f"Standard Deviation of K5: {std_k5_at_turn_9}")
print(f"Minimum K5: {min_k5_at_turn_9}")
print(f"Maximum K5: {max_k5_at_turn_9}")
print(f"Median K5: {median_k5_at_turn_9}")
print(f"Most Common K5 Values: {most_common_k5_at_turn_9}")
print(f"First 5 K5 Values: {first_5_k5_at_turn_9}")

# Print the results for Saldo Ops at Turn 10
print(f"--- Saldo Ops at Turn 10 Statistics ---")
print(f"Average Saldo Ops: {average_saldo_at_turn_9}")
print(f"Standard Deviation of Saldo Ops: {std_saldo_at_turn_9}")
print(f"Minimum Saldo Ops: {min_saldo_at_turn_9}")
print(f"Maximum Saldo Ops: {max_saldo_at_turn_9}")
print(f"Median Saldo Ops: {median_saldo_at_turn_9}")
print(f"Most Common Saldo Ops Values: {most_common_saldo_at_turn_9}")
print(f"First 5 Saldo Ops Values: {first_5_saldo_at_turn_9}")
# Export to Excel if output flag is set
# Select only t
# Select only the first 9 turns with iloc
df_saldo_ops_9turns = df_saldo_ops.iloc[:, :9]  # Select columns for turns 1 to 9
df_k5_9turns = df_k5.iloc[:, :9]  # Select columns for turns 1 to 9

# Export to Excel if output flag is set
if output == 1:
    with pd.ExcelWriter(output_file_path) as writer:
        df_saldo_ops_9turns.to_excel(writer, sheet_name='Saldo_Ops_9Turns')
        df_k5_9turns.to_excel(writer, sheet_name='K5_9Turns')
    print("DataFrames for 9 turns have been exported to Excel.")
else:
    print("Output flag is 0, so no export to Excel.")



