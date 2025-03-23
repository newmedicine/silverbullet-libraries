```space-lua

njb = njb or {}

function njb.isInTable(theTable, theObject) 
  for _,v in pairs(theTable) do
    if v == theObject then
      return true
    end
  end
  return false
end

function njb.getParentPaths(pPage)
  local results = {}
  local parent = string.match(pPage, "^(.*)/[^/]+$")
  while parent do
    table.insert(results, parent)
    parent = string.match(parent, "^(.*)/[^/]+$")
  end
  return results
end

function njb.navTreeButtons()
  
  local currentPageName = editor.getCurrentPath()
  print("currentPageName: " .. currentPageName)
  local parentPathString = njb.getParentPaths(currentPageName)[1]
  local upArrow = ""
  local leftArrow = ""
  local rightArrow = ""
  local pageWasRead = false
  
  if parentPathString then
    upArrow = "**[[" .. parentPathString .. "|⬆️ " .. parentPathString .. "]]**"
  else
    upArrow = "⬆️"
  end

  local pages = space.listPages()
  local pageNames = {} -- final list of just page names goes here

  for page in each(pages) do
    local parents = njb.getParentPaths(page.name)
    -- add the page itself
    if (parentPathString == parents[1]) and not njb.isInTable(pageNames, page.name) then
        table.insert(pageNames, page.name)
    end
    -- add the parent "pages"
    for i, path in ipairs(parents) do
        local parentParent = parents[i+1]
        if (parentPathString == parentParent) and (not njb.isInTable(pageNames, path)) then
            table.insert(pageNames, path)
        end
    end
    if(page.name == currentPageName) then 
      pageWasRead = true
    end
  end

  table.sort(pageNames, function(a, b)
    return string.lower(a) < string.lower(b)
    end)

  local previousPage = nil
  local insertionIndex = nil
  
  -- Find the first element that's greater than or equal to currentPageName (case-insensitive)
  for i, page in ipairs(pageNames) do
    -- print(i .. ". " .. page)
    if insertionIndex == nil and string.lower(page) >= string.lower(currentPageName) then
      insertionIndex = i
    end
  end
 
  -- If currentPageName would be placed at the end, set insertionIndex accordingly
  if not insertionIndex then
    insertionIndex = #pageNames + 1
  end
  -- If there's an element before the insertion index, that's our previous page
  if insertionIndex > 1 then
    previousPage = pageNames[insertionIndex - 1]
  end
  if previousPage then
    local lastSegment = string.match(previousPage, "([^/]+)$")
    leftArrow = "[[" .. previousPage .. "|⬅️ " .. lastSegment .. "]]"
  else
    leftArrow = "⬅️"
  end
 
  local nextPage = nil
  for i, page in ipairs(pageNames) do
    if page == currentPageName then
      if i < #pageNames then
        nextPage = pageNames[i + 1]
      end
      break
    end
  end
  if nextPage then
    local lastSegment = string.match(nextPage, "([^/]+)$")
    rightArrow = "[[" .. nextPage .. "| " .. lastSegment .. " ➡️]]"
  else
    rightArrow = "➡️"
  end
  
  local crlf = string.char(13) .. string.char(10)
  local folderFiles = ""

  print("pageWasRead "..pageWasRead)

  if pageWasRead == false then
    folderfiles = crlf
    for i,p in ipairs(pageNames) do
      folderFiles = folderFiles .. crlf .. "* [[" .. p .. "|" .. string.match(p, "([^/]+)$") .. "]] $mytag"
    end
  end

  return widget.new { markdown = leftArrow .. " | " .. upArrow .. " | " .. rightArrow .. folderFiles }
end
```

${njb.navTreeButtons()}

